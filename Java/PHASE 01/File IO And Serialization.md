## Overview

Every backend system reads and writes data to files, streams, and external formats. Your Spring Boot application will read configuration from YAML files, parse incoming JSON payloads, write audit logs to disk, process CSV uploads from clients, and serialize domain objects for storage and transmission. Java provides three generations of I/O APIs, and understanding which to use — and when — is essential for writing correct, efficient, and maintainable code. This section covers the modern `java.nio.file` API for file operations, stream-based I/O for reading and writing data, and the Jackson library for JSON serialization, which is the industry standard in Java backend development.

---

## Core Concepts

### Java I/O Evolution

Java's I/O capabilities have evolved through three major generations. Understanding all three helps you recognize legacy code and choose the right API for new code.

**Generation 1: `java.io` (Java 1.0, 1996)**

The original I/O library. Stream-based, blocking, and synchronous.

- Byte streams: `InputStream`, `OutputStream`
- Character streams: `Reader`, `Writer`
- File access: `java.io.File`

Still used internally by many libraries, but the `File` class has significant design flaws (no exception handling, inconsistent behavior across platforms, no symbolic link support). Avoid `java.io.File` in new code.

**Generation 2: `java.nio` (Java 1.4, 2002)**

New I/O. Introduced buffers, channels, and non-blocking I/O.

- `ByteBuffer`, `CharBuffer`
- `FileChannel`, `SocketChannel`
- `Selector` for multiplexed non-blocking I/O

Primarily used for high-performance networking and large file operations. You will encounter NIO channels when working with Kafka clients, Netty, and high-throughput file processing. For everyday file operations, NIO channels are unnecessarily complex.

**Generation 3: `java.nio.file` (Java 7, 2011)**

The modern file system API. Built on top of NIO but with a much cleaner interface.

- `Path` — represents a file system path (replacement for `java.io.File`)
- `Files` — static utility methods for file operations
- `FileSystem` — abstraction for different file systems (local, ZIP, in-memory)
- `WatchService` — file system event monitoring

This is the API you should use for all file operations in new code. It handles exceptions properly, supports symbolic links, provides atomic operations, and works consistently across operating systems.

### Path

`java.nio.file.Path` is the modern replacement for `java.io.File`. It represents a location in the file system.

```java
import java.nio.file.Path;
import java.nio.file.Paths;

// Creating paths
Path absolute = Path.of("/var/log/application.log");
Path relative = Path.of("src/main/resources/config.yml");
Path home = Path.of(System.getProperty("user.home"), ".myapp", "config.json");

// Legacy alternative (still works, but Path.of() is preferred since Java 11)
Path legacy = Paths.get("/var/log/application.log");

// Path components
Path path = Path.of("/home/john/projects/myapp/src/Main.java");
path.getFileName();    // "Main.java"
path.getParent();      // "/home/john/projects/myapp/src"
path.getRoot();        // "/"
path.getNameCount();   // 6
path.getName(0);       // "home"
path.getName(3);       // "myapp"
path.subpath(2, 5);    // "projects/myapp/src"

// Path operations
Path base = Path.of("/home/john/projects");
Path resolved = base.resolve("myapp/src/Main.java");
// "/home/john/projects/myapp/src/Main.java"

Path relativized = resolved.relativize(base);
// "../../.."

Path normalized = Path.of("/home/john/../john/./projects").normalize();
// "/home/john/projects"

// Path properties
path.isAbsolute();     // true
path.startsWith("/home");  // true
path.endsWith("Main.java"); // true

// Converting
path.toString();       // "/home/john/projects/myapp/src/Main.java"
path.toAbsolutePath(); // Resolves against current working directory
path.toRealPath();     // Resolves symbolic links and normalizes (throws if file does not exist)
path.toUri();          // "file:///home/john/projects/myapp/src/Main.java"
path.toFile();         // Legacy java.io.File (use only when interfacing with old APIs)
```

**Path vs File:**

| Feature | `java.io.File` | `java.nio.file.Path` |
|---------|---------------|---------------------|
| Exception handling | Returns boolean | Throws descriptive exceptions |
| Symbolic links | Limited support | Full support |
| Atomic operations | No | Yes |
| File attributes | Limited | Rich (permissions, timestamps, etc.) |
| File system abstraction | No | Yes |
| Watch service | No | Yes |
| Recommendation | Legacy only | Use for all new code |

### Files

`java.nio.file.Files` provides static methods for all common file operations. This is the workhorse class for file I/O.

**Reading files:**

```java
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;

Path path = Path.of("data/transactions.csv");

// Read entire file as a string (Java 11+)
String content = Files.readString(path);

// Read entire file as a string with specific charset
String utf8Content = Files.readString(path, StandardCharsets.UTF_8);

// Read all lines into a List<String>
List<String> lines = Files.readAllLines(path);
List<String> utf8Lines = Files.readAllLines(path, StandardCharsets.UTF_8);

// Read all bytes (for binary files)
byte[] bytes = Files.readAllBytes(path);

// Read lines as a Stream (lazy, memory-efficient for large files)
try (Stream<String> stream = Files.lines(path)) {
    stream.filter(line -> line.startsWith("TX-"))
          .map(Transaction::parse)
          .forEach(System.out::println);
}
// IMPORTANT: Files.lines() returns a Stream that holds an open file handle.
// You MUST close it with try-with-resources.
```

**Writing files:**

```java
Path outputPath = Path.of("output/report.json");

// Write a string (Java 11+)
Files.writeString(outputPath, "{\"status\": \"ok\"}");

// Write with specific options
Files.writeString(outputPath, content,
    StandardCharsets.UTF_8,
    StandardOpenOption.CREATE,
    StandardOpenOption.TRUNCATE_EXISTING
);

// Append to a file
Files.writeString(outputPath, "\nNew line",
    StandardOpenOption.CREATE,
    StandardOpenOption.APPEND
);

// Write lines
List<String> lines = List.of("line1", "line2", "line3");
Files.write(outputPath, lines);

// Write bytes
Files.write(outputPath, new byte[]{0x48, 0x65, 0x6C, 0x6C, 0x6F});
```

**File and directory operations:**

```java
Path file = Path.of("data/report.csv");
Path dir = Path.of("data/archive");

// Existence checks
Files.exists(file);
Files.notExists(file);
Files.isRegularFile(file);
Files.isDirectory(dir);
Files.isSymbolicLink(file);
Files.isReadable(file);
Files.isWritable(file);
Files.isExecutable(file);
Files.isHidden(file);

// File metadata
long size = Files.size(file);                    // Bytes
FileTime lastModified = Files.getLastModifiedTime(file);
FileTime created = Files.getAttribute(file, "creationTime", LinkOption.NOFOLLOW_LINKS);
String owner = Files.getOwner(file).getName();

// Create
Files.createFile(file);                          // Creates file, throws if exists
Files.createDirectory(dir);                      // Creates single directory
Files.createDirectories(Path.of("a/b/c/d"));     // Creates all parent directories
Files.createTempFile("prefix-", ".tmp");         // Creates temp file in default temp dir
Files.createTempDirectory("myapp-");             // Creates temp directory

// Copy
Files.copy(source, target);
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.copy(source, target, StandardCopyOption.COPY_ATTRIBUTES);
Files.copy(inputStream, target);                 // Copy from InputStream to file

// Move
Files.move(source, target);
Files.move(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);  // Atomic rename

// Delete
Files.delete(file);                              // Throws if file does not exist
Files.deleteIfExists(file);                      // Returns boolean, no exception

// Symbolic links
Files.createSymbolicLink(link, target);
Files.readSymbolicLink(link);
```

**Directory traversal:**

```java
Path root = Path.of("src/main/java");

// List immediate children
try (Stream<Path> entries = Files.list(root)) {
    entries.forEach(System.out::println);
}

// Walk the entire directory tree (recursive)
try (Stream<Path> tree = Files.walk(root)) {
    tree.filter(Files::isRegularFile)
        .filter(p -> p.toString().endsWith(".java"))
        .forEach(System.out::println);
}

// Walk with depth limit
try (Stream<Path> shallow = Files.walk(root, 2)) {
    shallow.forEach(System.out::println);
}

// Find files matching a condition
try (Stream<Path> matches = Files.find(root, 10,
        (path, attrs) -> attrs.isRegularFile()
            && path.toString().endsWith(".java")
            && attrs.size() > 10_000)) {
    matches.forEach(System.out::println);
}

// IMPORTANT: Files.list(), Files.walk(), and Files.find() all return
// Streams that hold open directory handles. Always use try-with-resources.
```

### Reading and Writing with Streams

For more control over I/O operations, use streams. Java distinguishes between byte streams (raw data) and character streams (text data).

**Byte streams (for binary data: images, PDFs, serialized objects):**

```java
// Reading bytes
Path file = Path.of("data/image.png");
try (InputStream in = Files.newInputStream(file)) {
    byte[] buffer = new byte[8192];
    int bytesRead;
    while ((bytesRead = in.read(buffer)) != -1) {
        processBytes(buffer, bytesRead);
    }
}

// Writing bytes
try (OutputStream out = Files.newOutputStream(file,
        StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING)) {
    out.write(new byte[]{1, 2, 3, 4, 5});
    out.flush();
}

// Buffered streams (reduces system calls, improves performance)
try (BufferedInputStream bis = new BufferedInputStream(Files.newInputStream(file))) {
    // Read operations are buffered — much faster for small reads
    int b = bis.read();
}

try (BufferedOutputStream bos = new BufferedOutputStream(Files.newOutputStream(file))) {
    // Write operations are buffered — much faster for small writes
    bos.write(42);
}
```

**Character streams (for text data: logs, CSV, JSON, config files):**

```java
// Reading text
Path file = Path.of("data/transactions.csv");
try (BufferedReader reader = Files.newBufferedReader(file, StandardCharsets.UTF_8)) {
    String line;
    while ((line = reader.readLine()) != null) {
        processLine(line);
    }
}

// Writing text
try (BufferedWriter writer = Files.newBufferedWriter(file, StandardCharsets.UTF_8,
        StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING)) {
    writer.write("id,amount,currency");
    writer.newLine();
    writer.write("TX-001,150.00,USD");
    writer.newLine();
}

// Bridge streams (convert between byte and character streams)
try (InputStream in = Files.newInputStream(file);
     InputStreamReader reader = new InputStreamReader(in, StandardCharsets.UTF_8);
     BufferedReader buffered = new BufferedReader(reader)) {
    String line = buffered.readLine();
}
```

**try-with-resources for I/O:**

Every I/O resource (stream, reader, writer, channel) must be closed after use. Failing to close resources causes file handle leaks, which can crash your application or exhaust the operating system's file descriptor limit.

```java
// WRONG — resource leak if an exception occurs between open and close
BufferedReader reader = Files.newBufferedReader(path);
String line = reader.readLine();
reader.close();  // Never reached if readLine() throws

// CORRECT — try-with-resources guarantees closure
try (BufferedReader reader = Files.newBufferedReader(path)) {
    String line = reader.readLine();
}  // reader.close() is called automatically, even if an exception occurs

// Multiple resources
try (BufferedReader reader = Files.newBufferedReader(inputPath);
     BufferedWriter writer = Files.newBufferedWriter(outputPath)) {
    String line;
    while ((line = reader.readLine()) != null) {
        writer.write(line.toUpperCase());
        writer.newLine();
    }
}  // Both reader and writer are closed automatically

// Resources are closed in reverse order of declaration
```

The `try-with-resources` statement works with any class that implements `AutoCloseable` (or its subinterface `Closeable`). This includes all streams, readers, writers, database connections, and HTTP connections.

### JSON Serialization with Jackson

Jackson is the de facto standard for JSON processing in Java. It is the default JSON library in Spring Boot and is used by nearly every enterprise Java application.

**Setup (Maven):**

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
    <version>2.17.0</version>
</dependency>
```

If you are using Spring Boot, Jackson is included automatically via `spring-boot-starter-web` or `spring-boot-starter-json`. You do not need to add these dependencies manually.

**Core class: ObjectMapper**

`ObjectMapper` is the main entry point for all Jackson operations. It is thread-safe after configuration and should be reused (not created per request).

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

// Create and configure once (typically as a Spring bean)
ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());  // Support for java.time types
mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);  // ISO-8601 format
mapper.enable(SerializationFeature.INDENT_OUTPUT);  // Pretty print (dev only)
```

**Serialization (Java object → JSON):**

```java
public record Transaction(
    String id,
    BigDecimal amount,
    String currency,
    LocalDateTime timestamp,
    TransactionStatus status
) {}

Transaction tx = new Transaction(
    "TX-001",
    new BigDecimal("1500.75"),
    "USD",
    LocalDateTime.of(2024, 3, 15, 10, 30, 0),
    TransactionStatus.COMPLETED
);

// To JSON string
String json = mapper.writeValueAsString(tx);
// {"id":"TX-001","amount":1500.75,"currency":"USD",
//  "timestamp":"2024-03-15T10:30:00","status":"COMPLETED"}

// To JSON file
mapper.writeValue(Path.of("output/tx.json").toFile(), tx);

// To JSON byte array (for HTTP responses)
byte[] bytes = mapper.writeValueAsBytes(tx);

// To a tree model (JsonNode)
JsonNode node = mapper.valueToTree(tx);
System.out.println(node.get("amount").asText());  // "1500.75"
```

**Deserialization (JSON → Java object):**

```java
String json = """
    {
        "id": "TX-002",
        "amount": 250.00,
        "currency": "EUR",
        "timestamp": "2024-03-16T14:00:00",
        "status": "PENDING"
    }
    """;

// From JSON string
Transaction tx = mapper.readValue(json, Transaction.class);

// From JSON file
Transaction fromFile = mapper.readValue(
    Path.of("data/tx.json").toFile(), Transaction.class
);

// From InputStream (e.g., HTTP request body)
Transaction fromStream = mapper.readValue(inputStream, Transaction.class);

// From byte array
Transaction fromBytes = mapper.readValue(bytes, Transaction.class);

// Deserializing a list
String jsonArray = """
    [{"id":"TX-001","amount":100,"currency":"USD","timestamp":"2024-01-01T00:00:00","status":"COMPLETED"},
     {"id":"TX-002","amount":200,"currency":"EUR","timestamp":"2024-01-02T00:00:00","status":"PENDING"}]
    """;

// TypeReference is required for generic types due to type erasure
List<Transaction> transactions = mapper.readValue(
    jsonArray,
    new TypeReference<List<Transaction>>() {}
);
```

**Jackson annotations:**

```java
public class Account {

    @JsonProperty("account_id")  // Map to a different JSON field name
    private String accountId;

    @JsonProperty("owner_name")
    private String ownerName;

    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "UTC")
    private LocalDateTime createdAt;

    @JsonIgnore  // Exclude from serialization and deserialization
    private String internalNotes;

    @JsonInclude(JsonInclude.Include.NON_NULL)  // Omit if null
    private String nickname;

    @JsonInclude(JsonInclude.Include.NON_EMPTY)  // Omit if null or empty
    private List<String> tags;

    // Constructor for deserialization
    @JsonCreator
    public Account(
        @JsonProperty("account_id") String accountId,
        @JsonProperty("owner_name") String ownerName
    ) {
        this.accountId = accountId;
        this.ownerName = ownerName;
        this.createdAt = LocalDateTime.now();
    }

    // Getters and setters...
}
```

**Key Jackson annotations:**

| Annotation | Purpose |
|-----------|---------|
| `@JsonProperty("name")` | Map field to a specific JSON property name |
| `@JsonIgnore` | Exclude field from serialization/deserialization |
| `@JsonInclude(Include.NON_NULL)` | Omit field if null |
| `@JsonFormat(pattern="...")` | Custom date/number format |
| `@JsonCreator` | Mark constructor or factory method for deserialization |
| `@JsonAlias("alt")` | Accept alternative JSON property names during deserialization |
| `@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)` | Apply naming strategy to all fields |
| `@JsonValue` | Serialize the entire object as a single value |
| `@JsonUnwrapped` | Flatten a nested object into the parent |
| `@JsonTypeInfo`, `@JsonSubTypes` | Polymorphic deserialization |

**Tree model (JsonNode):**

Use the tree model when you need to navigate JSON without mapping to a specific Java class.

```java
String json = """
    {
        "user": {
            "name": "Alice",
            "accounts": [
                {"id": "ACC-001", "balance": 5000},
                {"id": "ACC-002", "balance": 1200}
            ]
        }
    }
    """;

JsonNode root = mapper.readTree(json);

// Navigate
String name = root.get("user").get("name").asText();  // "Alice"
JsonNode accounts = root.get("user").get("accounts");
int count = accounts.size();  // 2

// Iterate
for (JsonNode account : accounts) {
    String id = account.get("id").asText();
    BigDecimal balance = account.get("balance").decimalValue();
    System.out.println(id + ": $" + balance);
}

// Check existence
if (root.has("user") && root.get("user").has("email")) {
    String email = root.get("user").get("email").asText();
}

// Modify (ObjectNode is mutable)
ObjectNode userNode = (ObjectNode) root.get("user");
userNode.put("email", "alice@example.com");
userNode.putArray("roles").add("admin").add("user");
```

**Custom serializers and deserializers:**

```java
// Custom serializer for BigDecimal (always 2 decimal places)
public class MoneySerializer extends JsonSerializer<BigDecimal> {
    @Override
    public void serialize(BigDecimal value, JsonGenerator gen,
                          SerializerProvider provider) throws IOException {
        gen.writeString(value.setScale(2, RoundingMode.HALF_UP).toPlainString());
    }
}

// Custom deserializer for BigDecimal (handles string and number inputs)
public class MoneyDeserializer extends JsonDeserializer<BigDecimal> {
    @Override
    public BigDecimal deserialize(JsonParser p, DeserializationContext ctxt)
            throws IOException {
        String text = p.getText();
        return new BigDecimal(text).setScale(2, RoundingMode.HALF_UP);
    }
}

// Usage
public record Payment(
    String id,
    @JsonSerialize(using = MoneySerializer.class)
    @JsonDeserialize(using = MoneyDeserializer.class)
    BigDecimal amount
) {}
```

### CSV Reading and Writing

CSV files are common in fintech for data imports, exports, and regulatory reporting.

**Using OpenCSV:**

```xml
<dependency>
    <groupId>com.opencsv</groupId>
    <artifactId>opencsv</artifactId>
    <version>5.9</version>
</dependency>
```

```java
import com.opencsv.CSVReader;
import com.opencsv.CSVWriter;
import com.opencsv.bean.CsvBindByName;
import com.opencsv.bean.CsvToBeanBuilder;
import com.opencsv.bean.StatefulBeanToCsvBuilder;

// Bean mapping
public class TransactionCsv {
    @CsvBindByName(column = "transaction_id")
    private String transactionId;

    @CsvBindByName(column = "amount")
    private BigDecimal amount;

    @CsvBindByName(column = "currency")
    private String currency;

    @CsvBindByName(column = "date")
    private String date;

    // Getters and setters required for OpenCSV
}

// Reading CSV to beans
try (Reader reader = Files.newBufferedReader(Path.of("data/transactions.csv"))) {
    List<TransactionCsv> transactions = new CsvToBeanBuilder<TransactionCsv>(reader)
        .withType(TransactionCsv.class)
        .withIgnoreLeadingWhiteSpace(true)
        .build()
        .parse();

    transactions.forEach(tx ->
        System.out.println(tx.getTransactionId() + ": " + tx.getAmount())
    );
}

// Writing beans to CSV
try (Writer writer = Files.newBufferedWriter(Path.of("output/report.csv"))) {
    new StatefulBeanToCsvBuilder<TransactionCsv>(writer)
        .build()
        .write(transactions);
}

// Low-level CSV reading (no bean mapping)
try (CSVReader reader = new CSVReader(
        Files.newBufferedReader(Path.of("data/raw.csv")))) {
    String[] header = reader.readNext();  // Read header row
    String[] row;
    while ((row = reader.readNext()) != null) {
        System.out.println(row[0] + " | " + row[1] + " | " + row[2]);
    }
}
```

### Properties Files

Properties files (`.properties`) are the traditional Java configuration format. Spring Boot supports them alongside YAML.

```properties
# application.properties
app.name=PaymentGateway
app.version=2.1.0
database.url=jdbc:postgresql://localhost:5432/payments
database.username=app_user
database.password=secret
database.pool.size=20
```

```java
import java.util.Properties;

// Reading properties
Properties props = new Properties();
try (InputStream in = Files.newInputStream(Path.of("config/application.properties"))) {
    props.load(in);
}

String appName = props.getProperty("app.name");  // "PaymentGateway"
int poolSize = Integer.parseInt(props.getProperty("database.pool.size", "10"));  // 20
String missing = props.getProperty("missing.key", "default");  // "default"

// Writing properties
Properties output = new Properties();
output.setProperty("app.name", "PaymentGateway");
output.setProperty("app.version", "2.2.0");
try (OutputStream out = Files.newOutputStream(Path.of("config/output.properties"))) {
    output.store(out, "Application Configuration");
}
```

In Spring Boot, you will rarely read properties files manually. Instead, use `@Value` or `@ConfigurationProperties` to inject values automatically. But understanding the underlying mechanism helps when debugging configuration issues.

### YAML with Jackson

YAML is the preferred configuration format for Spring Boot (`application.yml`). Jackson can read and write YAML with an additional module.

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-yaml</artifactId>
</dependency>
```

```java
import com.fasterxml.jackson.dataformat.yaml.YAMLMapper;

YAMLMapper yamlMapper = new YAMLMapper();

// Read YAML
String yaml = """
    server:
      port: 8080
      host: localhost
    database:
      url: jdbc:postgresql://localhost:5432/payments
      pool-size: 20
    """;

JsonNode config = yamlMapper.readTree(yaml);
int port = config.get("server").get("port").asInt();  // 8080

// Write YAML
ObjectNode root = yamlMapper.createObjectNode();
ObjectNode server = root.putObject("server");
server.put("port", 8080);
server.put("host", "localhost");

String yamlOutput = yamlMapper.writeValueAsString(root);
```

### Java Serialization (Understand but Avoid)

Java's built-in serialization (`java.io.Serializable`) converts objects to a byte stream and back. It is a legacy mechanism with serious security and compatibility problems.

```java
// How it works (DO NOT USE IN PRODUCTION)
public class LegacyTransaction implements Serializable {
    private static final long serialVersionUID = 1L;
    private String id;
    private BigDecimal amount;
    // ...
}

// Serialize
try (ObjectOutputStream oos = new ObjectOutputStream(
        Files.newOutputStream(Path.of("data/tx.ser")))) {
    oos.writeObject(transaction);
}

// Deserialize
try (ObjectInputStream ois = new ObjectInputStream(
        Files.newInputStream(Path.of("data/tx.ser")))) {
    LegacyTransaction tx = (LegacyTransaction) ois.readObject();
}
```

**Why you should avoid Java serialization:**

1. **Security vulnerabilities.** Deserialization of untrusted data can lead to remote code execution. This is one of the OWASP Top 10 vulnerabilities. Attackers can craft malicious byte streams that execute arbitrary code during deserialization.
2. **Tight coupling.** The serialized form is tied to the exact class structure. Adding a field, changing a type, or renaming a class breaks deserialization of old data.
3. **Performance.** Java serialization is slow and produces large byte streams compared to JSON, Protocol Buffers, or Avro.
4. **No cross-language support.** Only Java can deserialize Java-serialized objects. JSON, Avro, and Protocol Buffers work across all languages.

**Use instead:** JSON (Jackson), Protocol Buffers (gRPC), Avro (Kafka), or MessagePack for binary serialization.

The only place you will encounter `Serializable` in modern Java is as a marker interface required by some frameworks (e.g., JPA entities, Spring session attributes). Implement the interface but do not rely on the serialization mechanism.

---

## Code Examples

**A complete file processing pipeline for financial data:**

```java
package com.example.io;

import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

import java.io.*;
import java.math.BigDecimal;
import java.nio.file.*;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.*;
import java.util.stream.*;

public class FinancialDataProcessor {

    private final ObjectMapper mapper;

    public FinancialDataProcessor() {
        this.mapper = new ObjectMapper();
        this.mapper.registerModule(new JavaTimeModule());
        this.mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        this.mapper.enable(SerializationFeature.INDENT_OUTPUT);
    }

    // 1. Read transactions from CSV
    public List<Transaction> readFromCsv(Path csvPath) throws IOException {
        List<Transaction> transactions = new ArrayList<>();

        try (BufferedReader reader = Files.newBufferedReader(csvPath)) {
            String header = reader.readLine();  // Skip header
            String line;
            while ((line = reader.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields.length >= 4) {
                    transactions.add(new Transaction(
                        fields[0].trim(),
                        new BigDecimal(fields[1].trim()),
                        fields[2].trim(),
                        LocalDate.parse(fields[3].trim()),
                        TransactionStatus.valueOf(fields[4].trim())
                    ));
                }
            }
        }

        return transactions;
    }

    // 2. Write transactions to JSON
    public void writeToJson(List<Transaction> transactions, Path jsonPath)
            throws IOException {
        Files.createDirectories(jsonPath.getParent());
        mapper.writeValue(jsonPath.toFile(), transactions);
    }

    // 3. Read transactions from JSON
    public List<Transaction> readFromJson(Path jsonPath) throws IOException {
        return mapper.readValue(
            jsonPath.toFile(),
            new TypeReference<List<Transaction>>() {}
        );
    }

    // 4. Generate a summary report
    public void generateReport(List<Transaction> transactions, Path reportPath)
            throws IOException {
        Map<String, BigDecimal> byCategory = transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::currency,
                Collectors.reducing(
                    BigDecimal.ZERO,
                    Transaction::amount,
                    BigDecimal::add
                )
            ));

        try (BufferedWriter writer = Files.newBufferedWriter(reportPath)) {
            writer.write("=== Financial Summary Report ===");
            writer.newLine();
            writer.write("Generated: " + LocalDateTime.now());
            writer.newLine();
            writer.write("Total transactions: " + transactions.size());
            writer.newLine();
            writer.newLine();

            writer.write("Totals by currency:");
            writer.newLine();
            byCategory.forEach((currency, total) -> {
                try {
                    writer.write(String.format("  %s: %s", currency, total));
                    writer.newLine();
                } catch (IOException e) {
                    throw new UncheckedIOException(e);
                }
            });
        }
    }

    // 5. Process large files with streams (memory-efficient)
    public long countTransactionsAbove(Path csvPath, BigDecimal threshold)
            throws IOException {
        try (Stream<String> lines = Files.lines(csvPath)) {
            return lines.skip(1)  // Skip header
                .map(line -> line.split(","))
                .filter(fields -> fields.length >= 2)
                .map(fields -> new BigDecimal(fields[1].trim()))
                .filter(amount -> amount.compareTo(threshold) > 0)
                .count();
        }
    }

    public static void main(String[] args) throws IOException {
        FinancialDataProcessor processor = new FinancialDataProcessor();

        // Create sample CSV
        Path csvPath = Path.of("data/transactions.csv");
        Files.createDirectories(csvPath.getParent());
        Files.writeString(csvPath, """
            id,amount,currency,date,status
            TX-001,1500.00,USD,2024-01-15,COMPLETED
            TX-002,250.50,EUR,2024-01-16,COMPLETED
            TX-003,89.99,USD,2024-01-17,PENDING
            TX-004,3500.00,GBP,2024-01-18,COMPLETED
            TX-005,42.00,USD,2024-01-19,FAILED
            """);

        // Process
        List<Transaction> transactions = processor.readFromCsv(csvPath);
        System.out.println("Read " + transactions.size() + " transactions from CSV");

        Path jsonPath = Path.of("output/transactions.json");
        processor.writeToJson(transactions, jsonPath);
        System.out.println("Written to JSON: " + jsonPath);

        List<Transaction> fromJson = processor.readFromJson(jsonPath);
        System.out.println("Read " + fromJson.size() + " transactions from JSON");

        Path reportPath = Path.of("output/report.txt");
        processor.generateReport(transactions, reportPath);
        System.out.println("Report generated: " + reportPath);

        long largeCount = processor.countTransactionsAbove(
            csvPath, new BigDecimal("1000"));
        System.out.println("Transactions above $1000: " + largeCount);
    }

    public record Transaction(
        String id,
        BigDecimal amount,
        String currency,
        LocalDate date,
        TransactionStatus status
    ) {}

    public enum TransactionStatus {
        PENDING, COMPLETED, FAILED, CANCELLED
    }
}
```

---

## Important Notes

- Use `java.nio.file.Path` and `java.nio.file.Files` for all file operations in new code. Do not use `java.io.File`. The NIO.2 API provides better exception handling, symbolic link support, and atomic operations.
- Always use `try-with-resources` for I/O operations. Every stream, reader, writer, and channel holds an operating system file descriptor. Failing to close these resources causes file descriptor leaks that will eventually crash your application with "Too many open files" errors.
- `Files.readString()` and `Files.writeString()` (Java 11+) are the simplest way to read and write small-to-medium text files. For large files (hundreds of megabytes or more), use `Files.lines()` with streams to avoid loading the entire file into memory.
- `Files.lines()`, `Files.list()`, `Files.walk()`, and `Files.find()` return Streams that hold open file handles. You MUST close them with `try-with-resources`. Forgetting to close these streams is a common source of resource leaks.
- Jackson's `ObjectMapper` is thread-safe after configuration. Create one instance and reuse it across your application. In Spring Boot, an `ObjectMapper` bean is auto-configured and injected automatically. Do not create a new `ObjectMapper` per request — it is expensive to initialize.
- When deserializing generic types (e.g., `List<Transaction>`), you must use `TypeReference<List<Transaction>>()` because of type erasure. Jackson cannot determine the element type from `List.class` alone. This is a direct consequence of the type erasure discussed in the Generics section.
- Never use Java's built-in serialization (`ObjectOutputStream`/`ObjectInputStream`) for data that crosses trust boundaries. Deserialization of untrusted data is a critical security vulnerability (CVE-2015-7501 and many others). Use JSON, Protocol Buffers, or Avro instead.
- The `@JsonProperty` annotation is essential when your Java field names do not match the JSON property names. In fintech APIs, JSON often uses `snake_case` (e.g., `account_id`) while Java uses `camelCase` (e.g., `accountId`). Use `@JsonProperty("account_id")` or configure `PropertyNamingStrategies.SNAKE_CASE` globally.
- `BigDecimal` serialization requires careful handling. By default, Jackson serializes `BigDecimal` as a JSON number, which can lose precision for very large or very small values. Consider serializing as a string (`@JsonSerialize(using = ToStringSerializer.class)`) for financial amounts to preserve exact precision.
- When reading CSV files, always validate and sanitize the input. CSV files from external sources may contain malformed data, unexpected delimiters, or injection attempts. Never trust the structure of uploaded files.
- Character encoding matters. Always specify `StandardCharsets.UTF_8` explicitly when reading and writing text files. Relying on the platform default encoding causes bugs when your application runs on different operating systems or in Docker containers with different locale settings.
- File paths should be constructed using `Path.of()` or `Path.resolve()`, never by concatenating strings with `/` or `\`. String concatenation produces paths that break across operating systems. `Path.of("data", "output", "report.csv")` works correctly on macOS, Linux, and Windows.

---

## Practice

1. Write a program that reads a CSV file of transactions (id, amount, currency, date, status), filters out FAILED transactions, calculates the total amount per currency, and writes the results to a JSON file using Jackson. Use `Files.lines()` for memory-efficient reading.

2. Create a `Configuration` class with fields for `databaseUrl`, `databaseUsername`, `databasePassword`, `maxPoolSize`, and `enableSsl`. Serialize it to both JSON and YAML using Jackson. Then deserialize both files back to `Configuration` objects and verify they are equal.

3. Write a method that walks a directory tree and finds all `.java` files larger than 10 KB. For each file, count the number of lines, the number of methods (lines containing `public`, `private`, or `protected`), and the number of TODO comments. Write the results to a JSON report.

4. Demonstrate the resource leak problem. Open a `BufferedReader` without `try-with-resources` in a loop that runs 10,000 times. Observe the "Too many open files" error. Then fix it with `try-with-resources` and verify the error disappears.

5. Create a custom Jackson serializer and deserializer for a `Money` class that wraps `BigDecimal` and `String currency`. The JSON format should be `{"amount": "1500.75", "currency": "USD"}` with the amount always serialized as a string with exactly two decimal places.

6. Write a program that reads a large JSON file (generate one with 100,000 transaction records) using Jackson's streaming API (`JsonParser`) instead of `ObjectMapper.readValue()`. Compare the memory usage with the tree model approach.

7. In your Obsidian vault, create a decision table: "Which I/O approach should I use?" with rows for scenarios (small text file, large text file, binary file, JSON API payload, CSV import, configuration file, inter-service messaging) and columns recommending the specific API and library.

---

## References

- java.nio.file Package: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/package-summary.html
- Files API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/Files.html
- Path API: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/Path.html
- Jackson Documentation: https://github.com/FasterXML/jackson-docs
- Jackson Annotations: https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations
- OpenCSV Documentation: https://opencsv.sourceforge.net/
- "Effective Java" by Joshua Bloch — Item 9 (Prefer try-with-resources to try-finally), Item 89 (For instance control, prefer enum types to readResolve)
- OWASP Deserialization Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html
