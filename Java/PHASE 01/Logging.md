## Overview

Logging is the practice of recording events, states, and errors that occur during program execution. Unlike `System.out.println`, proper logging provides **structured, configurable, and level-based output** that can be routed to files, consoles, or external systems, filtered by severity, and enriched with contextual metadata like timestamps, thread names, and request IDs.

In production Java systems — especially in backend and fintech contexts — logging is non-negotiable. It is your primary tool for **debugging production issues**, **auditing behavior**, **tracing distributed requests**, and **detecting anomalies**. A system without proper logging is a black box you cannot operate.

The Java logging ecosystem is fragmented by history. Multiple frameworks exist (JUL, Log4j, Log4j2, Logback), which led to the creation of **facades** like SLF4J that let you write logging code once and swap implementations without changing application code.

---

## Core Concepts

### Why Logging Matters

```
- Debugging: understand what happened when things go wrong
- Auditing: prove what the system did (critical in fintech, compliance)
- Monitoring: feed logs into observability stacks (ELK, Splunk, Datadog)
- Performance analysis: measure durations, spot bottlenecks
- Security: detect suspicious activity, failed logins, unauthorized access
- Distributed tracing: follow a single request across microservices
```

`System.out.println` fails at all of these. It has no levels, no timestamps, no filtering, no rotation, no context. It also writes to stdout synchronously, blocking your thread and slowing your application under load.

### The Logging Facade Pattern

A **facade** is an abstraction layer that hides implementation details. **SLF4J** (Simple Logging Facade for Java) is the de-facto standard facade in the Java ecosystem.

```
Your Application Code
        │
        ▼
    SLF4J API (interface)
        │
        ▼
  ┌─────┴─────┬──────────┬─────────┐
  ▼           ▼          ▼         ▼
Logback   Log4j2     JUL      Simple
(binding) (binding) (binding) (binding)
```

**Why this matters:** You code against SLF4J. If you later want to swap Logback for Log4j2, you change dependencies, not code. Libraries you depend on also log through SLF4J, so all logs unify under one implementation.

### Log Levels

Log levels represent **severity**. They allow you to filter what gets recorded. Standard SLF4J levels, from least to most severe:

```
TRACE  → extremely detailed, usually for deep debugging
DEBUG  → diagnostic information for developers
INFO   → normal operational events (startup, shutdown, key state changes)
WARN   → something unexpected but recoverable
ERROR  → a failure that requires attention
```

You configure a **minimum level** per environment. Development might log at `DEBUG`; production usually logs at `INFO` or `WARN` to reduce volume.

### Parameterized Logging

Never use string concatenation in log calls:

```java
// BAD: builds the string even if DEBUG is disabled
log.debug("Processing user " + userId + " with balance " + balance);

// GOOD: SLF4J only builds the string if DEBUG is enabled
log.debug("Processing user {} with balance {}", userId, balance);
```

The `{}` placeholders defer string construction. This matters when disabled log statements are executed millions of times per second in hot code paths.

### Structured Logging

Traditional logs are plain text. **Structured logs** are machine-readable (usually JSON), with named fields:

```json
{
  "timestamp": "2025-01-15T10:23:45.123Z",
  "level": "INFO",
  "thread": "http-nio-8080-exec-1",
  "logger": "com.example.PaymentService",
  "message": "Payment processed",
  "userId": "u_12345",
  "amount": 150.00,
  "traceId": "abc123def456"
}
```

Structured logs are essential for aggregating and querying logs in tools like Elasticsearch, Datadog, or Splunk.

### MDC (Mapped Diagnostic Context)

MDC is a **thread-local map** that carries contextual data across log statements without passing it as parameters. It is ideal for request-scoped context like user IDs, request IDs, or trace IDs.

```java
MDC.put("requestId", "req-abc123");
MDC.put("userId", "u_42");
log.info("Processing payment");  // will include requestId and userId
MDC.clear();  // always clear after the request
```

Every log statement executed on that thread will automatically include those fields — critical for tracing a single request through many method calls.

---

## Code Examples

### Basic SLF4J Usage

**Maven dependencies (`pom.xml`):**

```xml
<dependencies>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.13</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.5.6</version>
    </dependency>
</dependencies>
```

`logback-classic` is Logback's SLF4J binding. Adding it automatically makes SLF4J route through Logback.

**Simple logger usage:**

```java
package com.example.finance;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class PaymentService {

    // Standard idiom: private, static, final, named after the class
    private static final Logger log = LoggerFactory.getLogger(PaymentService.class);

    public void processPayment(String userId, double amount) {
        log.info("Starting payment processing for user={} amount={}", userId, amount);

        try {
            if (amount <= 0) {
                log.warn("Rejected payment: non-positive amount user={} amount={}", userId, amount);
                return;
            }

            // ... processing logic ...
            log.debug("Validated payment details for user={}", userId);

            log.info("Payment completed successfully user={}", userId);
        } catch (Exception e) {
            // Pass the exception LAST — SLF4J will log the full stack trace
            log.error("Payment failed for user={} amount={}", userId, amount, e);
            throw e;
        }
    }
}
```

### All Log Levels

```java
log.trace("Entering method with args: {}", args);
log.debug("Cache miss for key={}", key);
log.info("Application started on port {}", port);
log.warn("Retry attempt {} of {}", attempt, maxAttempts);
log.error("Database connection failed", exception);
```

### Guarding Expensive Log Calls

Even with parameterized logging, if computing an argument is expensive, guard the call:

```java
if (log.isDebugEnabled()) {
    log.debug("Full transaction dump: {}", expensiveSerialization(txn));
}
```

### Logback Configuration (`logback.xml`)

Place in `src/main/resources/logback.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- Console appender: writes to stdout -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Rolling file appender: writes to disk, rotates daily and by size -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/application-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Package-level log configuration -->
    <logger name="com.example.finance" level="DEBUG"/>
    <logger name="org.springframework" level="WARN"/>
    <logger name="org.hibernate.SQL" level="DEBUG"/>

    <!-- Root logger: catches everything else -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>

</configuration>
```

### Pattern Layout Tokens

```
%d{pattern}    → timestamp
%thread        → thread name
%level / %-5level → log level (padded to 5 chars)
%logger{36}    → logger name, abbreviated to 36 chars
%msg           → the log message
%n             → newline
%X{key}        → MDC value for key
%mdc           → all MDC values
%ex / %throwable → exception stack trace
```

### JSON (Structured) Logging with Logback

Add the Logstash encoder:

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

```xml
<appender name="JSON_CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <includeMdcKeyName>requestId</includeMdcKeyName>
        <includeMdcKeyName>userId</includeMdcKeyName>
    </encoder>
</appender>
```

Every log line is now a single JSON object — ideal for shipping to Elasticsearch or Datadog.

### Using MDC for Request Tracing

```java
import org.slf4j.MDC;
import java.util.UUID;

public class RequestFilter {

    private static final Logger log = LoggerFactory.getLogger(RequestFilter.class);

    public void handleRequest(Request request) {
        String requestId = UUID.randomUUID().toString();
        MDC.put("requestId", requestId);
        MDC.put("userId", request.getUserId());

        try {
            log.info("Received request path={}", request.getPath());
            processRequest(request);
            log.info("Request completed");
        } catch (Exception e) {
            log.error("Request failed", e);
        } finally {
            // CRITICAL: always clear MDC to avoid leaking context to the next request
            // (threads are reused from a pool)
            MDC.clear();
        }
    }
}
```

Every log statement inside `handleRequest` (and any method it calls on the same thread) automatically includes `requestId` and `userId` in the output.

### Environment-Specific Configuration

Split configuration for dev vs prod using Logback's built-in property mechanisms or Spring profiles (in Spring Boot, name the file `logback-spring.xml`):

```xml
<configuration>
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>

    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="JSON_CONSOLE"/>
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>
</configuration>
```

### Log4j2 (Comparison)

Log4j2 is the main alternative to Logback. Similar concepts, different configuration format (`log4j2.xml`):

```xml
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

Key Log4j2 advantages:
- **Async logging** via LMAX Disruptor (much higher throughput under load)
- Better performance in high-volume scenarios
- More flexible plugin architecture

Logback remains the default in Spring Boot and is fine for the vast majority of applications.

---

## Important Notes

### Best Practices

- Use SLF4J API in all application code — never bind directly to Logback or Log4j2.
- Declare loggers as: `private static final Logger log = LoggerFactory.getLogger(ClassName.class)`
- Use parameterized logging (`{}`), never string concatenation.
- Pass exceptions as the LAST argument to log methods (SLF4J prints the stack trace).
- Choose the correct level: `TRACE` for deep internal diagnostics, `DEBUG` for developer diagnostics, `INFO` for operational milestones, `WARN` for recoverable anomalies, `ERROR` for failures needing attention.
- Never log at INFO in tight loops — you will drown the log system.
- Always clear MDC in a finally block.
- Use structured (JSON) logging in production for observability tooling.
- Rotate log files by size AND time; cap total disk usage.

### Security: What NEVER to Log

This is a critical fintech concern. Logs leak. Assume they will be read by attackers.

**Never log:**
- Passwords (plain or hashed)
- Full credit card numbers (only last 4 digits, if any)
- Social security numbers, government IDs
- API keys, tokens, session IDs
- Private encryption keys
- Full request bodies containing PII
- Full JWT tokens
- OAuth secrets

Mask sensitive data explicitly:

```java
log.info("User authenticated userId={} email={}", userId, maskEmail(email));

private String maskEmail(String email) {
    int at = email.indexOf('@');
    return email.charAt(0) + "***" + email.substring(at);
}
```

### Performance Considerations

- **Async appenders** (`AsyncAppender` in Logback, `AsyncLogger` in Log4j2) offload I/O to a background thread. Essential under high load.
- **Disabled log statements are cheap** — SLF4J short-circuits before formatting when the level is disabled.
- **Rolling policies must be configured** — unbounded log files fill disks and crash servers.
- **JSON logging has overhead** — usually acceptable, but benchmark for latency-critical services.

### Common Anti-Patterns

- `log.info("Entering method")` — too noisy, use DEBUG or do not log.
- `catch (Exception e) { log.error(e.getMessage()); }` — loses stack trace.
- `log.debug("data: " + expensiveToString())` — builds string even when disabled.
- `System.out.println("...")` — bypasses logging entirely, ungovernable.
- Logging the same event at multiple layers — duplicates confuse investigations.
- Using `printStackTrace()` — writes to stderr, no context, no format control.

### Correlation IDs and Distributed Tracing

In microservices architectures, a single user request traverses many services. A **correlation ID** (also called trace ID) is generated at the edge (API gateway or first service) and propagated via HTTP headers (`X-Request-Id`, `X-B3-TraceId`, or W3C `traceparent`) to every downstream service. Each service places it in MDC.

When aggregated in Elasticsearch/Datadog, you can filter by that ID and see the complete request journey across the entire system.

This connects directly to distributed tracing with tools like Zipkin, Jaeger, and OpenTelemetry, which are covered in the monitoring and observability phase of this roadmap.

---

## Practice

1. Set up a Maven project with SLF4J + Logback.
2. Configure both console and rolling file appenders.
3. Write a class with logs at every level (TRACE through ERROR).
4. Configure levels differently per package (your code DEBUG, third-party WARN).
5. Log an exception with full stack trace.
6. Implement a request handler that uses MDC to attach a request ID to every log.
7. Configure JSON structured logging with the Logstash encoder.
8. Simulate high log volume; observe file rotation triggering.
9. Write a masking utility for emails, card numbers, and API keys.
10. Refactor a class that uses `System.out.println` into proper SLF4J logging.
11. Compare Logback vs Log4j2 by benchmarking log throughput under load.
12. Configure environment-specific logging with Spring profiles (dev vs prod).

---

## References

- SLF4J Manual: https://www.slf4j.org/manual.html
- Logback Documentation: https://logback.qos.ch/documentation.html
- Log4j2 Documentation: https://logging.apache.org/log4j/2.x/
- Logstash Logback Encoder: https://github.com/logfellow/logstash-logback-encoder
- OWASP Logging Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html
