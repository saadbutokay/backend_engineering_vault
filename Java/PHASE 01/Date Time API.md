## Overview

Date and time handling is one of the most notoriously difficult problems in software engineering. Timezones, daylight saving time transitions, leap years, leap seconds, and calendar system variations make it a minefield of subtle bugs. In fintech, where every transaction has a timestamp, every trade has a settlement date, and every report spans fiscal periods, getting dates and times wrong is not a minor inconvenience—it is a regulatory and financial liability.

Java's original date/time API (`java.util.Date`, `java.util.Calendar`, `java.text.SimpleDateFormat`) is widely regarded as one of the worst API designs in the standard library. It is mutable, not thread-safe, confusingly named, and riddled with off-by-one errors. In Java 8, the entire API was replaced by `java.time` (JSR 310), a modern, immutable, thread-safe, and fluent API inspired by the Joda-Time library.

The rule is simple: **never use `java.util.Date` or `java.util.Calendar` in new code**. Use `java.time` exclusively. The only reason you will encounter the legacy API is when interfacing with old libraries or JDBC drivers that have not yet been updated—and even then, `java.time` provides conversion methods.

---

## Core Concepts

### Why the Legacy API Is Broken

**java.util.Date:**
- Despite the name, it represents a specific instant in time (date AND time), not just a date
- Months are 0-indexed (January = 0, December = 11) — a source of endless bugs
- Years are relative to 1900 (`new Date(2025, 0, 1)` creates January 1, 3925)
- Mutable: any code with a reference can change the value silently
- Not thread-safe: sharing a `Date` across threads causes data corruption
- Most constructors and methods are deprecated since Java 1.1 (1997)

**java.util.Calendar:**
- Also mutable and not thread-safe
- 0-indexed months (same trap)
- Verbose and unintuitive API (`calendar.set(Calendar.MONTH, Calendar.JANUARY)`)
- Lenient by default: `Calendar` accepts invalid dates and silently "fixes" them (January 32 becomes February 1)
- Performance overhead from timezone and locale calculations on every operation

**java.text.SimpleDateFormat:**
- **NOT thread-safe** — the single most common date-related bug in Java
- Sharing a `SimpleDateFormat` instance across threads causes corrupted output
- Workaround was `ThreadLocal<SimpleDateFormat>`, which is ugly and error-prone

### The java.time Design Philosophy

The `java.time` package (Java 8+) was designed with these principles:

1. **Immutable**: all classes are immutable and thread-safe — operations return new instances, never mutate the original
2. **Fluent API**: method chaining reads naturally — `date.plusDays(5).withMonth(3).withYear(2026)`
3. **Clear separation of concerns**:
   - Date without time (`LocalDate`)
   - Time without date (`LocalTime`)
   - Date and time without timezone (`LocalDateTime`)
   - Instant on the timeline (`Instant`)
   - Date and time with timezone (`ZonedDateTime`)
4. **Domain-driven naming**: class names describe exactly what they represent
5. **Null-safe**: methods throw clear exceptions instead of returning null
6. **Extensible**: `TemporalAdjusters`, `ChronoUnit`, and custom calendars

### The Core Type Hierarchy

```
                        Temporal
                           │
          ┌────────────────┼────────────────┐
          │                │                │
      TemporalAccessor  TemporalAmount   Chronology
          │                │
    ┌─────┴─────┐    ┌─────┴─────┐
    │           │    │           │
 LocalDate  LocalTime Duration  Period
    │
 LocalDateTime
    │
 ZonedDateTime
 OffsetDateTime

 Instant (standalone point on the timeline)
```

The key mental model:

```
LocalDate       → "2025-01-15"              (a date on a calendar, no time, no timezone)
LocalTime       → "14:30:00"                (a time on a clock, no date, no timezone)
LocalDateTime   → "2025-01-15T14:30:00"     (date + time, but WHERE? no timezone)
Instant         → "2025-01-15T19:30:00Z"    (a point on the global timeline, UTC)
ZonedDateTime   → "2025-01-15T14:30-05:00[America/New_York]"  (date + time + timezone)
OffsetDateTime  → "2025-01-15T14:30-05:00"  (date + time + UTC offset, no region rules)
```

### When to Use Which Type

**LocalDate:**
- Birthdays, holidays, settlement dates, fiscal period boundaries
- "What date is it?" without caring about the exact moment

**LocalTime:**
- Market opening/closing times, daily schedule entries
- "What time does the meeting start?" (in local context)

**LocalDateTime:**
- Logging events on a single server (where timezone is implicit)
- Scheduling within a single timezone context
- **CAUTION**: ambiguous across timezones and during DST transitions

**Instant:**
- Timestamps for events, audit logs, database records
- Measuring elapsed time between events
- Machine-to-machine communication
- **THE DEFAULT CHOICE** for recording "when something happened"

**ZonedDateTime:**
- Displaying times to users in their local timezone
- Scheduling across timezones ("3 PM New York time")
- Handling DST transitions correctly

**OffsetDateTime:**
- Storing timestamps with a known offset but no region rules
- Interfacing with systems that use fixed offsets
- Preferred over `ZonedDateTime` for database storage (SQL standard)

### Duration vs Period

Both represent "an amount of time," but they operate on different scales:

**Duration:**
- Time-based: hours, minutes, seconds, nanoseconds
- "How long did the API call take?"
- "Wait 30 seconds before retrying"
- Operates on `Instant`, `LocalDateTime`, `LocalTime`

**Period:**
- Date-based: years, months, days
- "How old is this account?"
- "The subscription renews every 3 months"
- Operates on `LocalDate`
- A `Period` of "1 month" means different numbers of days depending on the month

**Key difference:**
```
Duration.ofDays(1) = exactly 24 hours (86,400 seconds)
Period.ofDays(1)   = 1 calendar day (could be 23 or 25 hours during DST)
```

### TemporalAdjusters

`TemporalAdjusters` are pre-built date manipulation strategies. Instead of manually calculating "the last day of the month" or "the next Monday," you use a named adjuster:

```java
date.with(TemporalAdjusters.firstDayOfMonth());
date.with(TemporalAdjusters.lastDayOfYear());
date.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
```

### ChronoUnit

`ChronoUnit` is an enum representing units of time for calculating differences:

```java
long daysBetween = ChronoUnit.DAYS.between(startDate, endDate);
long hoursBetween = ChronoUnit.HOURS.between(startTime, endTime);
```

---

## Code Examples

### LocalDate — Working with Dates

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.DayOfWeek;

// Creating dates
LocalDate today = LocalDate.now();                          // 2025-01-15
LocalDate specific = LocalDate.of(2025, 1, 15);             // 2025-01-15
LocalDate fromMonth = LocalDate.of(2025, Month.JANUARY, 15); // same, more readable
LocalDate parsed = LocalDate.parse("2025-01-15");            // ISO-8601 format

// Extracting components
int year = today.getYear();              // 2025
int month = today.getMonthValue();       // 1 (1-indexed, unlike legacy API!)
Month monthEnum = today.getMonth();      // Month.JANUARY
int day = today.getDayOfMonth();         // 15
DayOfWeek dow = today.getDayOfWeek();    // DayOfWeek.WEDNESDAY
int dayOfYear = today.getDayOfYear();    // 15
int lengthOfMonth = today.lengthOfMonth(); // 31
boolean leapYear = today.isLeapYear();   // false

// Immutability: all operations return NEW instances
LocalDate tomorrow = today.plusDays(1);       // 2025-01-16
LocalDate nextMonth = today.plusMonths(1);    // 2025-02-15
LocalDate nextYear = today.plusYears(1);      // 2026-01-15
LocalDate lastWeek = today.minusWeeks(1);     // 2025-01-08

// today is STILL 2025-01-15 — it was not modified

// Chaining
LocalDate result = today
    .plusMonths(2)
    .minusDays(5)
    .withYear(2026);

// Comparison
LocalDate date1 = LocalDate.of(2025, 1, 15);
LocalDate date2 = LocalDate.of(2025, 3, 20);

date1.isBefore(date2);   // true
date1.isAfter(date2);    // false
date1.isEqual(date1);    // true
date1.compareTo(date2);  // negative (date1 < date2)

// Special dates
LocalDate epoch = LocalDate.ofEpochDay(0);       // 1970-01-01
LocalDate min = LocalDate.MIN;                    // -999999999-01-01
LocalDate max = LocalDate.MAX;                    // +999999999-12-31
```

### LocalTime — Working with Times

```java
import java.time.LocalTime;

// Creating times
LocalTime now = LocalTime.now();                  // 14:30:45.123456789
LocalTime specific = LocalTime.of(14, 30);        // 14:30:00
LocalTime withSeconds = LocalTime.of(14, 30, 45); // 14:30:45
LocalTime parsed = LocalTime.parse("14:30:45");   // ISO-8601

// Extracting components
int hour = now.getHour();         // 14
int minute = now.getMinute();     // 30
int second = now.getSecond();     // 45
int nano = now.getNano();         // 123456789

// Manipulation
LocalTime later = now.plusHours(2).plusMinutes(30);  // 17:00:45.123456789
LocalTime truncated = now.truncatedTo(ChronoUnit.MINUTES); // 14:30:00

// Special values
LocalTime midnight = LocalTime.MIDNIGHT;  // 00:00:00
LocalTime noon = LocalTime.NOON;          // 12:00:00
LocalTime min = LocalTime.MIN;            // 00:00:00
LocalTime max = LocalTime.MAX;            // 23:59:59.999999999
```

### LocalDateTime — Date and Time Without Timezone

```java
import java.time.LocalDateTime;
import java.time.LocalDate;
import java.time.LocalTime;

// Creating
LocalDateTime now = LocalDateTime.now();
LocalDateTime specific = LocalDateTime.of(2025, 1, 15, 14, 30, 0);
LocalDateTime combined = LocalDateTime.of(
    LocalDate.of(2025, 1, 15),
    LocalTime.of(14, 30)
);
LocalDateTime parsed = LocalDateTime.parse("2025-01-15T14:30:00");

// Extracting date and time parts
LocalDate datePart = now.toLocalDate();   // 2025-01-15
LocalTime timePart = now.toLocalTime();   // 14:30:00

// All the same manipulation methods as LocalDate and LocalTime
LocalDateTime tomorrow = now.plusDays(1);
LocalDateTime nextHour = now.plusHours(1);
```

**Important caveat:** `LocalDateTime` does not represent a specific instant on the timeline. `2025-01-15T14:30` in New York is a different moment than `2025-01-15T14:30` in Tokyo. Use it only when the timezone context is implicit and unambiguous.

### Instant — Machine Time (UTC)

```java
import java.time.Instant;
import java.time.Duration;

// Creating
Instant now = Instant.now();                              // 2025-01-15T19:30:00.123Z
Instant epoch = Instant.EPOCH;                            // 1970-01-01T00:00:00Z
Instant fromEpoch = Instant.ofEpochSecond(1736967000L);   // from Unix timestamp
Instant fromMillis = Instant.ofEpochMilli(1736967000000L); // from epoch millis
Instant parsed = Instant.parse("2025-01-15T19:30:00Z");   // ISO-8601

// Extracting
long epochSeconds = now.getEpochSecond();   // seconds since 1970-01-01T00:00:00Z
int nanos = now.getNano();                  // nanosecond-of-second
long millis = now.toEpochMilli();           // milliseconds since epoch

// Manipulation
Instant later = now.plusSeconds(3600);      // 1 hour later
Instant earlier = now.minus(Duration.ofMinutes(30));

// Comparison
Instant start = Instant.now();
// ... some operation ...
Instant end = Instant.now();

start.isBefore(end);  // true

// Measuring elapsed time
Duration elapsed = Duration.between(start, end);
System.out.println("Operation took " + elapsed.toMillis() + " ms");
```

`Instant` is the correct type for recording "when something happened." It is unambiguous, timezone-independent, and maps directly to Unix timestamps. Use it for audit logs, event timestamps, and database `TIMESTAMP WITH TIME ZONE` columns.

### ZonedDateTime — Date and Time with Timezone

```java
import java.time.ZonedDateTime;
import java.time.ZoneId;
import java.time.LocalDateTime;
import java.time.Instant;

// Creating
ZonedDateTime now = ZonedDateTime.now();                          // system default timezone
ZonedDateTime inNY = ZonedDateTime.now(ZoneId.of("America/New_York"));
ZonedDateTime specific = ZonedDateTime.of(
    2025, 1, 15, 14, 30, 0, 0,
    ZoneId.of("America/New_York")
);

// Converting from LocalDateTime (attaching a timezone)
LocalDateTime localDateTime = LocalDateTime.of(2025, 1, 15, 14, 30);
ZonedDateTime zoned = localDateTime.atZone(ZoneId.of("Europe/London"));
// 2025-01-15T14:30Z[Europe/London]

// Converting from Instant (instant → zoned)
Instant instant = Instant.now();
ZonedDateTime tokyoTime = instant.atZone(ZoneId.of("Asia/Tokyo"));

// Converting to Instant (zoned → instant, loses timezone info)
Instant backToInstant = zoned.toInstant();

// Timezone conversion (same instant, different wall-clock time)
ZonedDateTime nyTime = ZonedDateTime.of(2025, 6, 15, 9, 0, 0, 0,
    ZoneId.of("America/New_York"));
ZonedDateTime londonTime = nyTime.withZoneSameInstant(ZoneId.of("Europe/London"));
// nyTime:     2025-06-15T09:00-04:00[America/New_York]  (EDT)
// londonTime: 2025-06-15T14:00+01:00[Europe/London]      (BST)
// SAME instant, different wall-clock times

// withZoneSameLocal vs withZoneSameInstant:
ZonedDateTime sameLocal = nyTime.withZoneSameLocal(ZoneId.of("Europe/London"));
// 2025-06-15T09:00+01:00[Europe/London] — same wall-clock time, DIFFERENT instant
```

### OffsetDateTime — Date and Time with Fixed Offset

```java
import java.time.OffsetDateTime;
import java.time.ZoneOffset;

// Creating
OffsetDateTime now = OffsetDateTime.now();
OffsetDateTime specific = OffsetDateTime.of(2025, 1, 15, 14, 30, 0, 0,
    ZoneOffset.ofHours(-5));  // UTC-5

// Parsing
OffsetDateTime parsed = OffsetDateTime.parse("2025-01-15T14:30:00-05:00");

// Converting
Instant instant = parsed.toInstant();
ZonedDateTime zoned = parsed.toZonedDateTime();  // uses the offset as the zone
```

**OffsetDateTime vs ZonedDateTime:** `OffsetDateTime` stores a fixed UTC offset (`-05:00`). `ZonedDateTime` stores a region (`America/New_York`) which knows about DST rules. Use `OffsetDateTime` for database storage (SQL standard `TIMESTAMP WITH TIME ZONE` actually stores an offset, not a region). Use `ZonedDateTime` for business logic involving DST-aware scheduling.

### Duration — Time-Based Amounts

```java
import java.time.Duration;
import java.time.Instant;
import java.time.LocalTime;

// Creating
Duration thirtySeconds = Duration.ofSeconds(30);
Duration twoHours = Duration.ofHours(2);
Duration halfDay = Duration.ofDays(1).dividedBy(2);  // 12 hours
Duration fromString = Duration.parse("PT2H30M");       // ISO-8601: 2 hours 30 minutes
Duration between = Duration.between(
    LocalTime.of(9, 0),
    LocalTime.of(17, 30)
);  // PT8H30M

// Extracting components
long totalSeconds = twoHours.getSeconds();     // 7200
long totalMillis = twoHours.toMillis();        // 7200000
long totalMinutes = twoHours.toMinutes();      // 120
int hoursPart = between.toHoursPart();         // 8 (Java 9+)
int minutesPart = between.toMinutesPart();     // 30 (Java 9+)

// Manipulation
Duration longer = thirtySeconds.plusMinutes(5);  // PT5M30S
Duration shorter = twoHours.minus(Duration.ofMinutes(30));  // PT1H30M
Duration doubled = twoHours.multipliedBy(2);     // PT4H

// Comparison
thirtySeconds.isNegative();  // false
thirtySeconds.isZero();      // false
twoHours.compareTo(thirtySeconds);  // positive (twoHours > thirtySeconds)

// Use case: measuring operation duration
Instant start = Instant.now();
processPayment();
Instant end = Instant.now();
Duration elapsed = Duration.between(start, end);
log.info("Payment processed in {} ms", elapsed.toMillis());
```

### Period — Date-Based Amounts

```java
import java.time.Period;
import java.time.LocalDate;

// Creating
Period oneMonth = Period.ofMonths(1);
Period twoYears = Period.ofYears(2);
Period custom = Period.of(1, 6, 15);  // 1 year, 6 months, 15 days
Period fromString = Period.parse("P1Y6M15D");  // ISO-8601

// Calculating age
LocalDate birthDate = LocalDate.of(1990, 5, 20);
LocalDate today = LocalDate.of(2025, 1, 15);
Period age = Period.between(birthDate, today);

System.out.println("Age: " + age.getYears() + " years, "
    + age.getMonths() + " months, "
    + age.getDays() + " days");
// Age: 34 years, 7 months, 26 days

// Manipulation
Period longer = oneMonth.plusDays(10);  // P1M10D
Period doubled = oneMonth.multipliedBy(2);  // P2M

// IMPORTANT: Period.ofDays(30) != Period.ofMonths(1)
LocalDate jan1 = LocalDate.of(2025, 1, 1);
System.out.println(jan1.plus(Period.ofDays(30)));    // 2025-01-31
System.out.println(jan1.plus(Period.ofMonths(1)));   // 2025-02-01
// Different results! Days are exact; months follow calendar rules.

// Normalization (only normalizes days to months if days >= month length)
Period unnormalized = Period.of(0, 0, 45);
System.out.println(unnormalized);            // P45D (NOT normalized to P1M15D)
System.out.println(unnormalized.normalized()); // P1M15D (approximate, assumes 30-day months)
```

### DateTimeFormatter — Parsing and Formatting

```java
import java.time.format.DateTimeFormatter;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.ZonedDateTime;
import java.time.format.FormatStyle;
import java.util.Locale;

// --- PREDEFINED FORMATTERS ---

// ISO-8601 (default, no formatter needed)
LocalDate date = LocalDate.of(2025, 1, 15);
System.out.println(date.toString());  // "2025-01-15" (ISO_LOCAL_DATE)

LocalDateTime dateTime = LocalDateTime.of(2025, 1, 15, 14, 30);
System.out.println(dateTime.toString());  // "2025-01-15T14:30:00" (ISO_LOCAL_DATE_TIME)

// --- CUSTOM PATTERNS ---

DateTimeFormatter usDate = DateTimeFormatter.ofPattern("MM/dd/yyyy");
DateTimeFormatter europeanDate = DateTimeFormatter.ofPattern("dd.MM.yyyy");
DateTimeFormatter readable = DateTimeFormatter.ofPattern("MMMM d, yyyy");
DateTimeFormatter withTime = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
DateTimeFormatter full = DateTimeFormatter.ofPattern("EEEE, MMMM d, yyyy 'at' h:mm a");

// Formatting (object → string)
String formatted1 = date.format(usDate);       // "01/15/2025"
String formatted2 = date.format(europeanDate); // "15.01.2025"
String formatted3 = date.format(readable);     // "January 15, 2025"
String formatted4 = dateTime.format(withTime); // "2025-01-15 14:30:00"
String formatted5 = dateTime.format(full);     // "Wednesday, January 15, 2025 at 2:30 PM"

// Parsing (string → object)
LocalDate parsed1 = LocalDate.parse("01/15/2025", usDate);
LocalDate parsed2 = LocalDate.parse("15.01.2025", europeanDate);
LocalDateTime parsed3 = LocalDateTime.parse("2025-01-15 14:30:00", withTime);

// --- LOCALIZED FORMATTERS ---

DateTimeFormatter shortUS = DateTimeFormatter
    .ofLocalizedDate(FormatStyle.SHORT)
    .withLocale(Locale.US);
// "1/15/25"

DateTimeFormatter longGerman = DateTimeFormatter
    .ofLocalizedDate(FormatStyle.LONG)
    .withLocale(Locale.GERMANY);
// "15. Januar 2025"

DateTimeFormatter fullDateTime = DateTimeFormatter
    .ofLocalizedDateTime(FormatStyle.FULL)
    .withLocale(Locale.UK);

// --- COMMON PATTERN LETTERS ---
/*
  y  → year (yy = 25, yyyy = 2025)
  M  → month (M = 1, MM = 01, MMM = Jan, MMMM = January)
  d  → day of month (d = 5, dd = 05)
  E  → day of week (E = Wed, EEEE = Wednesday)
  H  → hour 0-23 (HH = 14)
  h  → hour 1-12 (hh = 02)
  m  → minute (mm = 30)
  s  → second (ss = 45)
  S  → fraction of second (SSS = milliseconds)
  a  → AM/PM
  z  → timezone name (EST, PST)
  Z  → timezone offset (+0500)
  X  → ISO-8601 timezone offset (+05, +05:00, Z)
  '  → escape for literal text ('at' → "at")
*/

// --- THREAD SAFETY ---
// DateTimeFormatter is IMMUTABLE and THREAD-SAFE.
// You can safely store it as a static final constant:
public class DateFormats {
    public static final DateTimeFormatter ISO_DATE = DateTimeFormatter.ISO_LOCAL_DATE;
    public static final DateTimeFormatter US_DATE = DateTimeFormatter.ofPattern("MM/dd/yyyy");
    public static final DateTimeFormatter TIMESTAMP = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    // No ThreadLocal needed! (unlike SimpleDateFormat)
}
```

### ZoneId and ZoneOffset

```java
import java.time.ZoneId;
import java.time.ZoneOffset;
import java.util.Set;

// ZoneId: a named timezone region with DST rules
ZoneId ny = ZoneId.of("America/New_York");
ZoneId london = ZoneId.of("Europe/London");
ZoneId tokyo = ZoneId.of("Asia/Tokyo");
ZoneId systemDefault = ZoneId.systemDefault();

// List all available timezone IDs
Set<String> allZones = ZoneId.getAvailableZoneIds();
System.out.println(allZones.size());  // ~600 zones

// ZoneOffset: a fixed offset from UTC (no DST rules)
ZoneOffset utc = ZoneOffset.UTC;                    // Z
ZoneOffset plusFive = ZoneOffset.ofHours(5);        // +05:00
ZoneOffset minusFiveThirty = ZoneOffset.ofHoursMinutes(-5, -30); // -05:30
ZoneOffset fromString = ZoneOffset.of("+05:30");    // +05:30

// Getting the current offset for a zone (accounts for DST)
ZoneId newYork = ZoneId.of("America/New_York");
ZoneOffset winterOffset = newYork.getRules().getOffset(
    java.time.Instant.parse("2025-01-15T12:00:00Z"));  // -05:00 (EST)
ZoneOffset summerOffset = newYork.getRules().getOffset(
    java.time.Instant.parse("2025-07-15T12:00:00Z"));  // -04:00 (EDT)
```

### TemporalAdjusters — Date Manipulation Strategies

```java
import java.time.LocalDate;
import java.time.DayOfWeek;
import java.time.temporal.TemporalAdjusters;

LocalDate date = LocalDate.of(2025, 1, 15); // Wednesday

// First and last
date.with(TemporalAdjusters.firstDayOfMonth());       // 2025-01-01
date.with(TemporalAdjusters.lastDayOfMonth());        // 2025-01-31
date.with(TemporalAdjusters.firstDayOfYear());        // 2025-01-01
date.with(TemporalAdjusters.lastDayOfYear());         // 2025-12-31
date.with(TemporalAdjusters.firstDayOfNextMonth());   // 2025-02-01
date.with(TemporalAdjusters.firstDayOfNextYear());    // 2026-01-01

// Day of week
date.with(TemporalAdjusters.next(DayOfWeek.MONDAY));      // 2025-01-20
date.with(TemporalAdjusters.previous(DayOfWeek.FRIDAY));  // 2025-01-10
date.with(TemporalAdjusters.nextOrSame(DayOfWeek.WEDNESDAY)); // 2025-01-15 (same day)
date.with(TemporalAdjusters.previousOrSame(DayOfWeek.MONDAY)); // 2025-01-13

// Ordinal day of week in month
date.with(TemporalAdjusters.dayOfWeekInMonth(2, DayOfWeek.TUESDAY));
// 2nd Tuesday of January 2025 → 2025-01-14

// Custom adjuster: next business day (skip weekends)
TemporalAdjuster nextBusinessDay = temporal -> {
    LocalDate d = LocalDate.from(temporal);
    do {
        d = d.plusDays(1);
    } while (d.getDayOfWeek() == DayOfWeek.SATURDAY
          || d.getDayOfWeek() == DayOfWeek.SUNDAY);
    return d;
};

LocalDate nextBiz = date.with(nextBusinessDay);  // 2025-01-16 (Thursday)

// Fintech example: end of quarter
TemporalAdjuster endOfQuarter = temporal -> {
    LocalDate d = LocalDate.from(temporal);
    int quarterEndMonth = ((d.getMonthValue() - 1) / 3 + 1) * 3;
    return d.withMonth(quarterEndMonth).with(TemporalAdjusters.lastDayOfMonth());
};

LocalDate q1End = LocalDate.of(2025, 2, 15).with(endOfQuarter);  // 2025-03-31
```

### ChronoUnit — Calculating Differences

```java
import java.time.temporal.ChronoUnit;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.Instant;

LocalDate start = LocalDate.of(2025, 1, 1);
LocalDate end = LocalDate.of(2025, 12, 31);

long days = ChronoUnit.DAYS.between(start, end);       // 364
long weeks = ChronoUnit.WEEKS.between(start, end);     // 52
long months = ChronoUnit.MONTHS.between(start, end);   // 11
long years = ChronoUnit.YEARS.between(start, end);     // 0

LocalDateTime morning = LocalDateTime.of(2025, 1, 15, 9, 0);
LocalDateTime evening = LocalDateTime.of(2025, 1, 15, 17, 30);

long hours = ChronoUnit.HOURS.between(morning, evening);     // 8
long minutes = ChronoUnit.MINUTES.between(morning, evening); // 510

// Truncating
LocalDateTime precise = LocalDateTime.of(2025, 1, 15, 14, 37, 45, 123456789);
precise.truncatedTo(ChronoUnit.DAYS);    // 2025-01-15T00:00:00
precise.truncatedTo(ChronoUnit.HOURS);   // 2025-01-15T14:00:00
precise.truncatedTo(ChronoUnit.MINUTES); // 2025-01-15T14:37:00

// Checking support
start.isSupported(ChronoUnit.HOURS);  // false (LocalDate doesn't support time units)
morning.isSupported(ChronoUnit.HOURS); // true
```

### Converting Between Legacy and Modern API

```java
import java.time.Instant;
import java.time.ZoneId;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Date;
import java.util.Calendar;
import java.sql.Timestamp;

// --- java.util.Date ↔ Instant ---
Date legacyDate = new Date();
Instant instant = legacyDate.toInstant();              // Date → Instant
Date backToDate = Date.from(instant);                  // Instant → Date

// --- java.util.Calendar ↔ ZonedDateTime ---
Calendar calendar = Calendar.getInstance();
ZonedDateTime zdt = calendar.toInstant()
    .atZone(calendar.getTimeZone().toZoneId());        // Calendar → ZonedDateTime
// Calendar ← ZonedDateTime: no direct method, convert via Instant

// --- java.sql.Timestamp ↔ Instant ---
Timestamp timestamp = Timestamp.from(Instant.now());   // Instant → Timestamp
Instant fromTs = timestamp.toInstant();                // Timestamp → Instant

// --- java.sql.Date ↔ LocalDate ---
java.sql.Date sqlDate = java.sql.Date.valueOf(LocalDate.now());  // LocalDate → sql.Date
LocalDate localDate = sqlDate.toLocalDate();                      // sql.Date → LocalDate

// --- java.sql.Time ↔ LocalTime ---
java.sql.Time sqlTime = java.sql.Time.valueOf(java.time.LocalTime.now());
java.time.LocalTime localTime = sqlTime.toLocalTime();
```

### Working with Timestamps in Databases (JDBC)

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.OffsetDateTime;

// Modern JDBC (4.2+) supports java.time types directly

// WRITING to database
String insertSql = "INSERT INTO transactions (amount, created_at, settlement_date) VALUES (?, ?, ?)";
try (PreparedStatement ps = connection.prepareStatement(insertSql)) {
    ps.setBigDecimal(1, new BigDecimal("150.00"));
    ps.setObject(2, Instant.now());                    // TIMESTAMP WITH TIME ZONE
    ps.setObject(3, LocalDate.now());                  // DATE
    ps.executeUpdate();
}

// READING from database
String selectSql = "SELECT created_at, settlement_date FROM transactions WHERE id = ?";
try (PreparedStatement ps = connection.prepareStatement(selectSql)) {
    ps.setLong(1, 42L);
    try (ResultSet rs = ps.executeQuery()) {
        if (rs.next()) {
            Instant createdAt = rs.getObject("created_at", Instant.class);
            LocalDate settlementDate = rs.getObject("settlement_date", LocalDate.class);
        }
    }
}

// For TIMESTAMP WITH TIME ZONE columns, prefer OffsetDateTime:
OffsetDateTime odt = rs.getObject("created_at", OffsetDateTime.class);
```

### Working with Timestamps in Databases (JPA/Hibernate)

```java
import jakarta.persistence.*;
import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.OffsetDateTime;

@Entity
@Table(name = "transactions")
public class Transaction {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // Best choice for audit timestamps: Instant
    // Maps to TIMESTAMP WITH TIME ZONE in PostgreSQL
    @Column(name = "created_at", nullable = false, updatable = false)
    private Instant createdAt;

    // Best choice for business dates: LocalDate
    // Maps to DATE in PostgreSQL
    @Column(name = "settlement_date")
    private LocalDate settlementDate;

    // Acceptable for local timestamps: LocalDateTime
    // Maps to TIMESTAMP (without timezone) — USE WITH CAUTION
    @Column(name = "local_event_time")
    private LocalDateTime localEventTime;

    // SQL-standard way to store timezone-aware timestamps
    // Maps to TIMESTAMP WITH TIME ZONE
    @Column(name = "processed_at")
    private OffsetDateTime processedAt;

    @PrePersist
    protected void onCreate() {
        this.createdAt = Instant.now();
        this.processedAt = OffsetDateTime.now();
    }
}
```

---

## Important Notes

### The Golden Rule: Store UTC, Display Local

This is the single most important principle in date/time handling, especially in fintech:

```
STORE:   Always store timestamps in UTC (Instant or OffsetDateTime with UTC offset)
         Database column type: TIMESTAMP WITH TIME ZONE (PostgreSQL stores it as UTC internally)
         Java type: Instant or OffsetDateTime

DISPLAY: Convert to the user's local timezone only at the presentation layer
         Java type: ZonedDateTime (for DST-aware conversion)

NEVER:   Store LocalDateTime in the database for event timestamps
         It has no timezone information and is ambiguous during DST transitions
```

**Why this matters in fintech:**
- A trade executed at 9:30 AM New York time and a trade at 9:30 AM London time are different instants
- Regulatory reports require precise, unambiguous timestamps
- Audit trails must be comparable across global offices
- Daylight saving time transitions cause `LocalDateTime` to be ambiguous (the same wall-clock time occurs twice in autumn)

### DST Transition Pitfalls

```java
// Spring forward: March 10, 2024 at 2:00 AM → 3:00 AM (EST → EDT)
// The time 2:30 AM does not exist on this day in New York
ZonedDateTime springForward = ZonedDateTime.of(
    2024, 3, 10, 2, 30, 0, 0, ZoneId.of("America/New_York"));
// Result: 2024-03-10T03:30-04:00[America/New_York] (auto-adjusted forward)

// Fall back: November 3, 2024 at 2:00 AM → 1:00 AM (EDT → EST)
// The time 1:30 AM occurs TWICE on this day
ZonedDateTime fallBack = ZonedDateTime.of(
    2024, 11, 3, 1, 30, 0, 0, ZoneId.of("America/New_York"));
// Result: 2024-11-03T01:30-04:00[America/New_York] (first occurrence, EDT)
// The second occurrence would be 2024-11-03T01:30-05:00[America/New_York] (EST)
```

`ZonedDateTime` handles these transitions correctly. `LocalDateTime` does not know they exist.

### Common Mistakes

1. **Using `LocalDateTime` for event timestamps** — Use `Instant` or `OffsetDateTime`. `LocalDateTime` is ambiguous across timezones.

2. **Using `double`/`float` for durations** — Use `Duration`. Floating-point arithmetic loses precision.

3. **Using `Period` and `Duration` interchangeably** — `Period` = calendar days/months/years. `Duration` = exact seconds. `Period.ofDays(1)` during DST spring-forward = 23 hours, not 24.

4. **Parsing dates without specifying a formatter** — `LocalDate.parse("01/15/2025")` throws `DateTimeParseException`. Always specify the formatter for non-ISO formats.

5. **Sharing `SimpleDateFormat` across threads** — This is a legacy problem. `DateTimeFormatter` is thread-safe. Just use `java.time`.

6. **Using `==` to compare dates** — Use `.equals()` or `.isEqual()`. `==` compares object references.

7. **Forgetting that months are 1-indexed in `java.time`** — `Month.JANUARY = 1`, not 0. (Unlike the legacy `Calendar` API.)

8. **Storing timezone names in the database** — Store the UTC instant. The timezone is a display concern. Exception: if you need to know the original timezone for future re-calculation, store the timezone ID in a separate column.

9. **Using `Instant.now()` for business dates** — `Instant` represents a moment in time, not a calendar date. For "today's date" in a specific timezone: `LocalDate.now(ZoneId.of("America/New_York"))`. `Instant.now()` at 11 PM UTC on Jan 15 is already Jan 16 in Tokyo.

10. **Ignoring leap seconds** — Java's `Instant` does not account for leap seconds. For most applications, this is fine. For astronomical or ultra-precise scientific systems, it matters.

### Fintech-Specific Date/Time Patterns

**Trade timestamps:**
- `Instant` (UTC, nanosecond precision)
- Store as `TIMESTAMP WITH TIME ZONE` in PostgreSQL

**Settlement dates:**
- `LocalDate` (T+1, T+2 settlement)
- Store as `DATE` in PostgreSQL
- Use `TemporalAdjusters` to skip weekends and holidays

**Market hours:**
- `LocalTime` for open/close (e.g., 09:30, 16:00)
- `ZonedDateTime` to determine if market is currently open
- Account for early close days and holidays

**Fiscal periods:**
- `YearMonth` for monthly reporting
- Custom `TemporalAdjuster` for quarter boundaries
- `LocalDate` for period start/end

**Audit trails:**
- `Instant` for "when" (immutable, UTC)
- `OffsetDateTime` if the original offset must be preserved
- Never update timestamps — append-only

**SLA calculations:**
- `Duration` for response time SLAs (e.g., "respond within 30 seconds")
- `Period` for contractual periods (e.g., "30-day notice period")
- Business-day calculators that skip weekends and holidays

### Useful Additional Types

```java
import java.time.YearMonth;
import java.time.Year;
import java.time.MonthDay;
import java.time.DayOfWeek;

// YearMonth: useful for credit card expiry, fiscal months
YearMonth expiry = YearMonth.of(2027, 12);     // 2027-12
YearMonth current = YearMonth.now();            // 2025-01
expiry.isAfter(current);                        // true
expiry.lengthOfMonth();                         // 31
expiry.atDay(1);                                // 2027-12-01 (first day)
expiry.atEndOfMonth();                          // 2027-12-31 (last day)

// Year: useful for fiscal year comparisons
Year fiscalYear = Year.of(2025);
fiscalYear.isLeap();                            // false
fiscalYear.length();                            // 365

// MonthDay: useful for birthdays, annual events
MonthDay birthday = MonthDay.of(Month.MARCH, 15);
birthday.atYear(2025);                          // 2025-03-15

// DayOfWeek enum
DayOfWeek.MONDAY.getValue();                    // 1 (ISO-8601: Monday = 1)
DayOfWeek.SUNDAY.getValue();                    // 7
```

---

## Practice

1. Create a `LocalDate` for your birthday and calculate your exact age using `Period.between()`
2. Parse the string `"15/01/2025"` into a `LocalDate` using a custom `DateTimeFormatter`
3. Format the current date as `"Wednesday, January 15, 2025"` using a custom pattern
4. Calculate the number of business days (Mon-Fri) between two dates using a loop and `DayOfWeek`
5. Write a method that returns the last business day of a given month (skip weekends)
6. Convert an `Instant` to a `ZonedDateTime` in three different timezones and print all three
7. Demonstrate the DST spring-forward gap: create a `ZonedDateTime` for 2:30 AM on March 10, 2024 in `America/New_York` and observe the auto-adjustment
8. Calculate the `Duration` between two `Instants` and express it in hours, minutes, and seconds
9. Write a method that takes a `LocalDate` and returns the start and end dates of its quarter
10. Create a custom `TemporalAdjuster` that finds the next payday (15th or last business day of month)
11. Demonstrate that `DateTimeFormatter` is thread-safe by parsing dates from 10 concurrent threads
12. Write a JPA entity with `createdAt` (`Instant`), `updatedAt` (`Instant`), and `settlementDate` (`LocalDate`)
13. Compare `Period.ofMonths(1)` vs `Period.ofDays(30)` when added to January 1 vs February 1
14. Write a method that checks if a given `ZonedDateTime` falls within NYSE trading hours (9:30 AM - 4:00 PM ET, Mon-Fri)
15. Build a simple age calculator CLI: input a birthdate string, output years/months/days

---

## References

- Oracle Java Time API Tutorial: https://docs.oracle.com/javase/tutorial/datetime/
- JSR 310 Specification: https://jcp.org/en/jsr/detail?id=310
- java.time Javadoc: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/package-summary.html
- "Java SE 8 Date and Time" — Ben Evans (Oracle)
- Time Zone Database (IANA): https://www.iana.org/time-zones
- PostgreSQL Date/Time Types: https://www.postgresql.org/docs/current/datatype-datetime.html
