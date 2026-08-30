---
title: "Java - File I/O - FileReader FileWriter BufferedReader"
phase: "Phase 1 - Java Core and OOP"
language: "java"
tags:
  - backend
  - java
  - oop
  - io
  - file-io
  - nio
  - streams
status: "not-started"
---


# Java - File I/O - FileReader FileWriter BufferedReader

> [!abstract] Overview
> File I/O is the mechanism by which a Java program reads data from and writes data to files on the filesystem. Java provides two generations of I/O APIs: the legacy `java.io` package (streams, readers, writers) introduced in Java 1.0, and the modern `java.nio.file` package (Paths, Files) introduced in Java 7. In backend development, file I/O is used for reading configuration files, processing CSV and JSON data imports, writing log files, generating reports, handling file uploads, and managing temporary files. Understanding both the legacy and modern APIs is essential because you will encounter both in production codebases.

---

## Prerequisites

> [!info] Before reading this note, you should understand:
> - [[Java - Basic Input Output - Scanner and System.out]]
> - [[Java - Strings and String Methods]]
> - [[Java - Exception Handling - Try Catch Finally Throw Throws]]
> - [[Java - Java 8 Streams API]] (for the Files.lines() section)

---

## Theory

### The Java I/O Architecture

Java's I/O system is built on the concept of **streams**: abstract representations of data flows. There are two parallel hierarchies:

**Byte streams** (`java.io.InputStream` / `OutputStream`): Handle raw binary data one byte at a time. Use these for non-text data: images, audio, video, PDFs, serialized objects.

**Character streams** (`java.io.Reader` / `Writer`): Handle text data one character at a time, automatically handling character encoding (UTF-8, UTF-16, etc.). Use these for text files: CSV, JSON, XML, log files, configuration files.

```text
Byte Streams (binary data)          Character Streams (text data)
InputStream                         Reader
├── FileInputStream                 ├── FileReader
├── BufferedInputStream             ├── BufferedReader
├── ByteArrayInputStream            ├── InputStreamReader
└── ObjectInputStream               ├── StringReader
                                    └── CharArrayReader

OutputStream                        Writer
├── FileOutputStream                ├── FileWriter
├── BufferedOutputStream            ├── BufferedWriter
├── ByteArrayOutputStream           ├── OutputStreamWriter
└── ObjectOutputStream              ├── PrintWriter
                                    └── StringWriter
```

### The Legacy `java.io` API

**`FileReader`**: A convenience class for reading character files. It wraps a `FileInputStream` and an `InputStreamReader` with the platform's default character encoding.

**`FileWriter`**: A convenience class for writing character files. It wraps a `FileOutputStream` and an `OutputStreamWriter` with the platform's default character encoding.

**`BufferedReader`**: Wraps another `Reader` and adds a buffer (default 8192 characters) to reduce the number of I/O operations. Reading one character at a time from a file is extremely slow because each read triggers a system call to the operating system. The buffer reads a large block of characters in one system call and serves subsequent reads from memory.

**`BufferedWriter`**: Wraps another `Writer` and adds a buffer for efficient writing.

**`PrintWriter`**: A convenience writer that provides `print()`, `println()`, and `printf()` methods, similar to `System.out`. It is the most commonly used writer for generating text output.

### The Modern `java.nio.file` API (Java 7+)

The `java.nio.file` package (NIO.2) provides a higher-level, more convenient API for file operations. The two key classes are:

**`Path`**: Represents a file or directory path. Created using `Path.of("path/to/file.txt")` (Java 11+) or `Paths.get("path/to/file.txt")` (Java 7+).

**`Files`**: A utility class with static methods for all common file operations: reading, writing, copying, moving, deleting, checking existence, and more. Most methods accept a `Path` argument.

**Why prefer NIO.2 over legacy I/O:**

1. **Convenience**: `Files.readString()` reads an entire file in one line. The legacy approach requires creating a `FileReader`, wrapping it in a `BufferedReader`, reading line by line, and closing the reader.
2. **Encoding control**: NIO.2 methods accept a `Charset` parameter. `FileReader` and `FileWriter` use the platform default encoding, which varies between operating systems and causes cross-platform bugs.
3. **Exception handling**: NIO.2 methods throw specific exceptions (`NoSuchFileException`, `AccessDeniedException`) instead of the generic `IOException`.
4. **Stream integration**: `Files.lines()` returns a `Stream<String>`, enabling functional processing of file contents.

### Character Encoding

Character encoding is the mapping between characters and bytes. The most common encodings are:

| Encoding | Description | Use Case |
|----------|-------------|----------|
| UTF-8 | Variable-width (1-4 bytes per character). Backward compatible with ASCII. | Default for web, JSON, XML, modern applications |
| UTF-16 | Variable-width (2-4 bytes per character). | Java's internal String representation |
| ISO-8859-1 | Single-byte (Latin-1). | Legacy Western European systems |
| US-ASCII | Single-byte (7-bit). | Legacy English-only systems |

**The encoding trap**: `FileReader` and `FileWriter` use the platform's default encoding, which is UTF-8 on most modern Linux and Mac systems but may be Windows-1252 on older Windows systems. If you write a file on Windows with `FileWriter` and read it on Linux with `FileReader`, non-ASCII characters (like Bangla text) will be corrupted. Always specify the encoding explicitly using `InputStreamReader`/`OutputStreamWriter` or the NIO.2 API.

### Resource Management

File handles are operating system resources. If you open a file and forget to close it, the file handle leaks. On Linux, each process has a limit on open file descriptors (typically 1024 by default). Exceeding this limit causes `IOException: Too many open files`, which crashes the application.

**Three ways to manage file resources:**

1. **Try-with-resources** (Java 7+, recommended): Automatically closes the resource when the block exits, even if an exception occurs.
2. **Finally block** (legacy): Manually close the resource in a `finally` block.
3. **Explicit close**: Call `close()` directly. Risky because an exception before the `close()` call will leak the resource.

### How File I/O Works Internally

When your Java program reads a file, the following happens:

1. **System call**: The JVM calls the operating system's `open()` system call to get a file descriptor.
2. **Buffer allocation**: If using a buffered reader, the JVM allocates a byte array (default 8192 bytes) in heap memory.
3. **Data transfer**: The OS reads data from the disk (or page cache) into the JVM's buffer via the `read()` system call. Disk I/O is orders of magnitude slower than memory access, which is why buffering is critical.
4. **Character decoding**: For character streams, the raw bytes are decoded into Java `char` values using the specified encoding.
5. **Delivery**: The decoded characters are returned to your application code.

**Performance hierarchy** (fastest to slowest):
- Reading from memory (buffer): ~1 nanosecond per byte
- Reading from SSD: ~100 microseconds per operation
- Reading from HDD: ~10 milliseconds per operation
- Reading over network: ~1-100 milliseconds per operation

This is why buffering matters: a single 8KB buffer read from an SSD takes ~100 microseconds, while 8192 individual byte reads would take ~800 milliseconds (8000x slower).

> [!tip] Key Insight
> In modern Spring Boot backend development, you will rarely use `FileReader` or `FileWriter` directly. The NIO.2 `Files` class handles most file operations in a single line. However, understanding the legacy API is important because: (1) you will encounter it in older codebases, (2) the `BufferedReader` / `BufferedWriter` pattern is still the most efficient approach for processing very large files line by line, and (3) the stream-based architecture underpins all Java I/O, including network sockets and HTTP connections.

---

## Syntax and Basic Examples

### Example 1: Reading a file with the legacy API

```java
import java.io.*;

public class LegacyReadDemo {
    public static void main(String[] args) {
        // Method 1: FileReader (reads one character at a time, SLOW)
        try (FileReader reader = new FileReader("data.txt")) {
            int charCode;
            while ((charCode = reader.read()) != -1) {  // -1 means end of file
                System.out.print((char) charCode);
            }
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }

        // Method 2: BufferedReader (reads in chunks, FAST)
        try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
            String line;
            while ((line = reader.readLine()) != null) {  // null means end of file
                System.out.println(line);
            }
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }

        // Method 3: BufferedReader with explicit encoding (RECOMMENDED for legacy API)
        try (BufferedReader reader = new BufferedReader(
                new InputStreamReader(new FileInputStream("data.txt"), "UTF-8"))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }
    }
}
```

### Example 2: Writing a file with the legacy API

```java
import java.io.*;

public class LegacyWriteDemo {
    public static void main(String[] args) {
        // Method 1: FileWriter (overwrites the file)
        try (FileWriter writer = new FileWriter("output.txt")) {
            writer.write("Hello, Backend World!\n");
            writer.write("This is line 2.\n");
        } catch (IOException e) {
            System.out.println("Error writing file: " + e.getMessage());
        }

        // Method 2: FileWriter in append mode
        try (FileWriter writer = new FileWriter("output.txt", true)) {
            writer.write("This line is appended.\n");
        } catch (IOException e) {
            System.out.println("Error writing file: " + e.getMessage());
        }

        // Method 3: BufferedWriter (buffered, efficient for many writes)
        try (BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
            writer.write("Line 1");
            writer.newLine();  // Platform-independent newline
            writer.write("Line 2");
            writer.newLine();
        } catch (IOException e) {
            System.out.println("Error writing file: " + e.getMessage());
        }

        // Method 4: PrintWriter (convenient print/println/printf methods)
        try (PrintWriter writer = new PrintWriter(
                new BufferedWriter(new FileWriter("output.txt")))) {
            writer.println("Product Report");
            writer.println("================");
            writer.printf("%-15s %10s %8s%n", "Product", "Price", "Stock");
            writer.printf("%-15s %10.2f %8d%n", "Laptop", 85000.0, 15);
            writer.printf("%-15s %10.2f %8d%n", "Mouse", 1500.0, 50);
        } catch (IOException e) {
            System.out.println("Error writing file: " + e.getMessage());
        }
    }
}
```

### Example 3: Reading and writing with NIO.2 (modern API)

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.io.IOException;
import java.util.List;

public class NioDemo {
    public static void main(String[] args) throws IOException {
        Path filePath = Path.of("data.txt");

        // Write entire content in one call
        String content = "Line 1: Dhaka\nLine 2: Sylhet\nLine 3: Rajshahi\n";
        Files.writeString(filePath, content, StandardCharsets.UTF_8);
        // Java 11+. For Java 7-10, use:
        // Files.write(filePath, content.getBytes(StandardCharsets.UTF_8));

        // Read entire content in one call
        String readContent = Files.readString(filePath, StandardCharsets.UTF_8);
        System.out.println("Content:\n" + readContent);

        // Read all lines into a List
        List<String> lines = Files.readAllLines(filePath, StandardCharsets.UTF_8);
        System.out.println("Lines: " + lines);
        System.out.println("Line count: " + lines.size());

        // Write a list of lines
        List<String> newLines = List.of("Apple", "Banana", "Cherry");
        Files.write(Path.of("fruits.txt"), newLines, StandardCharsets.UTF_8);

        // Append to a file
        Files.writeString(filePath, "Line 4: Khulna\n",
            StandardCharsets.UTF_8, StandardOpenOption.APPEND);

        // Check file properties
        System.out.println("Exists: " + Files.exists(filePath));
        System.out.println("Size: " + Files.size(filePath) + " bytes");
        System.out.println("Is readable: " + Files.isReadable(filePath));
        System.out.println("Is writable: " + Files.isWritable(filePath));
    }
}
```

### Example 4: Processing large files line by line

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.io.*;
import java.util.stream.Stream;

public class LargeFileDemo {
    public static void main(String[] args) {
        Path csvPath = Path.of("orders.csv");

        // Method 1: Files.lines() returns a Stream<String> (Java 8+)
        // The file is read lazily, one line at a time. Memory-efficient for large files.
        try (Stream<String> lines = Files.lines(csvPath, StandardCharsets.UTF_8)) {
            long orderCount = lines
                .skip(1)  // Skip the header row
                .filter(line -> !line.isBlank())
                .filter(line -> line.contains("PAID"))
                .count();

            System.out.println("Paid orders: " + orderCount);
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }

        // Method 2: BufferedReader for maximum control (best for very large files)
        try (BufferedReader reader = Files.newBufferedReader(csvPath, StandardCharsets.UTF_8)) {
            String header = reader.readLine();  // Read and skip header
            String line;
            int lineCount = 0;
            double totalRevenue = 0;

            while ((line = reader.readLine()) != null) {
                lineCount++;
                String[] fields = line.split(",");
                if (fields.length >= 4) {
                    try {
                        double amount = Double.parseDouble(fields[3].strip());
                        totalRevenue += amount;
                    } catch (NumberFormatException e) {
                        System.out.println("Invalid amount on line " + (lineCount + 1));
                    }
                }
            }

            System.out.printf("Processed %d lines. Total revenue: %,.2f BDT%n",
                lineCount, totalRevenue);
        } catch (IOException e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

### Example 5: File and directory operations

```java
import java.nio.file.*;
import java.io.IOException;
import java.util.stream.Stream;

public class FileOperationsDemo {
    public static void main(String[] args) throws IOException {
        Path dir = Path.of("reports");

        // Create directories (including parent directories)
        Files.createDirectories(dir);

        // Create a file
        Path reportFile = dir.resolve("monthly-report.txt");
        if (!Files.exists(reportFile)) {
            Files.createFile(reportFile);
        }

        // Copy a file
        Path backupFile = dir.resolve("monthly-report-backup.txt");
        Files.copy(reportFile, backupFile, StandardCopyOption.REPLACE_EXISTING);

        // Move/rename a file
        Path movedFile = dir.resolve("archived-report.txt");
        Files.move(backupFile, movedFile, StandardCopyOption.REPLACE_EXISTING);

        // List files in a directory
        System.out.println("Files in reports/:");
        try (Stream<Path> files = Files.list(dir)) {
            files.filter(Files::isRegularFile)
                .forEach(path -> System.out.println("  " + path.getFileName()));
        }

        // Walk a directory tree recursively
        System.out.println("\nAll files recursively:");
        try (Stream<Path> allFiles = Files.walk(Path.of("."))) {
            allFiles.filter(Files::isRegularFile)
                .filter(path -> path.toString().endsWith(".java"))
                .limit(10)
                .forEach(path -> System.out.println("  " + path));
        }

        // Delete a file
        Files.deleteIfExists(movedFile);

        // Get file attributes
        System.out.println("\nFile info:");
        System.out.println("  Size: " + Files.size(reportFile) + " bytes");
        System.out.println("  Last modified: " + Files.getLastModifiedTime(reportFile));
    }
}
```

### Example 6: CSV parsing and generation

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.io.*;
import java.util.*;

public class CsvDemo {

    record Student(String name, String department, double cgpa, int semester) {}

    public static void main(String[] args) throws IOException {
        // Writing a CSV file
        List<Student> students = List.of(
            new Student("Saad", "CSE", 3.72, 6),
            new Student("Rahim", "EEE", 3.45, 5),
            new Student("Karim", "CSE", 3.90, 7)
        );

        Path csvPath = Path.of("students.csv");
        try (BufferedWriter writer = Files.newBufferedWriter(csvPath, StandardCharsets.UTF_8)) {
            writer.write("Name,Department,CGPA,Semester");
            writer.newLine();
            for (Student s : students) {
                writer.write(String.format("%s,%s,%.2f,%d",
                    s.name(), s.department(), s.cgpa(), s.semester()));
                writer.newLine();
            }
        }

        // Reading and parsing the CSV file
        List<Student> parsedStudents = new ArrayList<>();
        try (BufferedReader reader = Files.newBufferedReader(csvPath, StandardCharsets.UTF_8)) {
            String header = reader.readLine();  // Skip header
            String line;
            while ((line = reader.readLine()) != null) {
                String[] fields = line.split(",");
                if (fields.length == 4) {
                    parsedStudents.add(new Student(
                        fields[0].strip(),
                        fields[1].strip(),
                        Double.parseDouble(fields[2].strip()),
                        Integer.parseInt(fields[3].strip())
                    ));
                }
            }
        }

        System.out.println("Parsed " + parsedStudents.size() + " students:");
        parsedStudents.forEach(System.out::println);
    }
}
```

---

## Real Backend Code

> [!example] How this appears in actual backend projects
> File I/O in Spring Boot backends is typically handled through higher-level abstractions, but the underlying principles are the same. Here are three realistic scenarios.

### Scenario 1: Processing uploaded files in a Spring Boot controller

When a client uploads a file through a multipart HTTP request, Spring Boot provides the file as a `MultipartFile` object. You can read its contents using the same stream-based patterns.

```java
package com.company.orderservice.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;

@RestController
@RequestMapping("/api/v1/import")
public class ImportController {

    private final ImportService importService;

    public ImportController(ImportService importService) {
        this.importService = importService;
    }

    @PostMapping("/products")
    public ResponseEntity<ImportResponse> importProducts(
            @RequestParam("file") MultipartFile file) {

        // Validate the uploaded file
        if (file.isEmpty()) {
            throw new ValidationException("file", "Uploaded file is empty");
        }
        if (!file.getOriginalFilename().endsWith(".csv")) {
            throw new ValidationException("file", "Only CSV files are accepted");
        }
        if (file.getSize() > 10 * 1024 * 1024) {  // 10 MB limit
            throw new ValidationException("file", "File size exceeds 10 MB limit");
        }

        try {
            // Read the file content using a BufferedReader
            // MultipartFile.getInputStream() returns an InputStream,
            // which we wrap in a BufferedReader for efficient line-by-line reading.
            try (BufferedReader reader = new BufferedReader(
                    new InputStreamReader(file.getInputStream(), StandardCharsets.UTF_8))) {

                ImportResult result = importService.processCsvImport(reader);
                return ResponseEntity.ok(new ImportResponse(
                    result.imported(), result.skipped(), result.errors()
                ));
            }

        } catch (IOException e) {
            throw new AppException("Failed to read uploaded file", 500, "FILE_READ_ERROR", e);
        }
    }
}
```

**What to notice:**

- `file.getInputStream()` returns a standard Java `InputStream`. The same `InputStreamReader` + `BufferedReader` pattern from the legacy API works here. This demonstrates that the stream-based I/O model is universal: whether the data comes from a file on disk, an HTTP upload, a network socket, or a database BLOB, the reading pattern is the same.
- The `try-with-resources` block ensures that the input stream is closed even if an exception occurs during processing. Failing to close the stream would leak the file handle and the temporary file that Spring creates for the upload.
- File validation (size, type, emptiness) happens before any I/O. This prevents wasting resources on invalid uploads.

### Scenario 2: Generating and downloading reports

Backend systems frequently generate reports (CSV, PDF, Excel) and serve them as file downloads.

```java
package com.company.orderservice.service;

import org.springframework.stereotype.Service;
import java.io.BufferedWriter;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;

@Service
public class ReportService {

    private final OrderRepository orderRepository;

    public ReportService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    // Generate a CSV report and write it to a temporary file
    public Path generateMonthlyReport(int year, int month) throws IOException {
        List<Order> orders = orderRepository.findByYearAndMonth(year, month);

        // Create a temporary file that will be deleted when the JVM exits
        Path tempFile = Files.createTempFile(
            "report-" + year + "-" + month + "-", ".csv"
        );

        // Write the report using BufferedWriter for efficiency
        try (BufferedWriter writer = Files.newBufferedWriter(tempFile, StandardCharsets.UTF_8)) {
            // Header row
            writer.write("Order Number,Date,Customer,Total,Status");
            writer.newLine();

            // Data rows
            for (Order order : orders) {
                writer.write(String.format("%s,%s,%s,%.2f,%s",
                    escapeCsv(order.getOrderNumber()),
                    order.getCreatedAt().toLocalDate(),
                    escapeCsv(order.getCustomerName()),
                    order.getTotalAmount().doubleValue(),
                    order.getStatus().name()
                ));
                writer.newLine();
            }
        }

        return tempFile;
    }

    // Helper: escape CSV fields that contain commas, quotes, or newlines
    private String escapeCsv(String value) {
        if (value == null) return "";
        if (value.contains(",") || value.contains("\"") || value.contains("\n")) {
            return "\"" + value.replace("\"", "\"\"") + "\"";
        }
        return value;
    }
}
```

```java
// Controller that serves the report as a download:
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import java.io.IOException;
import java.nio.file.Path;

@GetMapping("/reports/monthly")
public ResponseEntity<Resource> downloadMonthlyReport(
        @RequestParam int year, @RequestParam int month) throws IOException {

    Path reportPath = reportService.generateMonthlyReport(year, month);

    Resource resource = new UrlResource(reportPath.toUri());
    String filename = "orders-" + year + "-" + month + ".csv";

    return ResponseEntity.ok()
        .contentType(MediaType.parseMediaType("text/csv"))
        .header(HttpHeaders.CONTENT_DISPOSITION,
            "attachment; filename=\"" + filename + "\"")
        .body(resource);
}
```

**What to notice:**

- `Files.createTempFile()` creates a temporary file in the system's temp directory. This avoids cluttering the application's working directory with generated files.
- The `BufferedWriter` writes the CSV data efficiently. For a report with 100,000 rows, the buffer reduces the number of disk writes from 100,000 to approximately 12 (assuming 8KB buffer and ~100 bytes per row).
- The `escapeCsv()` method handles the CSV edge case where field values contain commas, quotes, or newlines. Without proper escaping, the generated CSV would be malformed.

### Scenario 3: Reading configuration from a properties file

```java
package com.company.orderservice.config;

import java.io.*;
import java.nio.file.*;
import java.util.Properties;

public class LegacyConfigLoader {

    // This pattern is common in older Java applications and libraries.
    // Spring Boot's @ConfigurationProperties replaces this for most use cases,
    // but you will encounter it in legacy code and third-party libraries.
    public Properties loadProperties(String configPath) {
        Properties props = new Properties();
        Path path = Path.of(configPath);

        if (!Files.exists(path)) {
            System.out.println("Config file not found: " + configPath);
            return props;  // Return empty properties
        }

        // Try-with-resources ensures the stream is closed
        try (InputStream input = Files.newInputStream(path)) {
            props.load(input);
            // Properties.load() reads key=value pairs from the input stream.
            // It handles comments (# and !), multi-line values (\), and
            // Unicode escapes (\uXXXX) automatically.
        } catch (IOException e) {
            throw new RuntimeException("Failed to load config: " + configPath, e);
        }

        return props;
    }

    public void saveProperties(Properties props, String configPath) {
        try (OutputStream output = Files.newOutputStream(Path.of(configPath))) {
            props.store(output, "Application Configuration");
            // Properties.store() writes key=value pairs with a timestamp comment.
        } catch (IOException e) {
            throw new RuntimeException("Failed to save config: " + configPath, e);
        }
    }
}
```

**What to notice:**

- `Files.newInputStream()` and `Files.newOutputStream()` are the NIO.2 equivalents of `FileInputStream` and `FileOutputStream`. They provide the same functionality with better exception handling and encoding support.
- The `Properties` class is a specialized `Map<Object, Object>` that reads and writes `.properties` files. It is one of the oldest Java APIs (Java 1.0) and is still widely used.
- In modern Spring Boot, you would use `@ConfigurationProperties` or `@Value` instead of manually loading properties files. But understanding the underlying I/O mechanism helps you debug configuration issues.

---

## Common Mistakes

> [!warning] Mistakes beginners make with this concept

### Mistake 1: Not closing file resources (resource leak)

**Wrong:**

```java
BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
String line = reader.readLine();
System.out.println(line);
// Forgot to close! The file handle leaks.
// If this code runs in a loop, the application will eventually crash
// with "Too many open files".
```

**Right:**

```java
// Try-with-resources: automatically closes the reader
try (BufferedReader reader = new BufferedReader(new FileReader("data.txt"))) {
    String line = reader.readLine();
    System.out.println(line);
} catch (IOException e) {
    System.out.println("Error: " + e.getMessage());
}
```

**Why it is wrong:** Every open file consumes an operating system file descriptor. If you do not close files, the descriptors accumulate until the process hits the OS limit (typically 1024 on Linux). At that point, all file operations fail with `IOException: Too many open files`. This is one of the most common causes of production outages in Java backends. Always use try-with-resources.

### Mistake 2: Using FileReader/FileWriter without specifying encoding

**Wrong:**

```java
// Uses the platform default encoding (varies by OS and locale)
BufferedReader reader = new BufferedReader(new FileReader("data.txt"));
// On Windows with a Bangla locale, this might use Windows-1252.
// On Linux, it might use UTF-8.
// The same code produces different results on different machines!
```

**Right:**

```java
// Explicit UTF-8 encoding: consistent across all platforms
BufferedReader reader = Files.newBufferedReader(
    Path.of("data.txt"), StandardCharsets.UTF_8
);

// Or with the legacy API:
BufferedReader reader = new BufferedReader(
    new InputStreamReader(new FileInputStream("data.txt"), StandardCharsets.UTF_8)
);
```

**Why it is wrong:** `FileReader` and `FileWriter` use the platform's default character encoding, which is determined by the operating system and locale settings. A file written on a developer's Mac (UTF-8) may be unreadable on a production Windows server (Windows-1252) if the encoding is not specified explicitly. Always use UTF-8 and specify it explicitly.

### Mistake 3: Reading an entire large file into memory

**Wrong:**

```java
// Reads the ENTIRE file into a single String.
// A 2 GB log file will cause OutOfMemoryError.
String content = Files.readString(Path.of("huge-log.txt"));
```

**Right:**

```java
// Process the file line by line. Memory usage is constant regardless of file size.
try (Stream<String> lines = Files.lines(Path.of("huge-log.txt"), StandardCharsets.UTF_8)) {
    long errorCount = lines
        .filter(line -> line.contains("ERROR"))
        .count();
    System.out.println("Error count: " + errorCount);
}

// Or with BufferedReader for maximum control:
try (BufferedReader reader = Files.newBufferedReader(
        Path.of("huge-log.txt"), StandardCharsets.UTF_8)) {
    String line;
    while ((line = reader.readLine()) != null) {
        processLine(line);  // Process one line at a time
    }
}
```

**Why it is wrong:** `Files.readString()` and `Files.readAllLines()` load the entire file into memory. For small files (a few MB), this is fine. For large files (hundreds of MB or GB), it causes `OutOfMemoryError` and crashes the JVM. Always process large files line by line using `Files.lines()` (streaming) or `BufferedReader.readLine()` (iterative).

### Mistake 4: Using `File` instead of `Path` (legacy API)

**Wrong:**

```java
// The java.io.File class is the legacy way to represent file paths.
// It has many design flaws: inconsistent error handling (returns boolean
// instead of throwing exceptions), no support for symbolic links,
// platform-dependent path separators, and no encoding control.
File file = new File("data/output.txt");
if (file.exists()) {
    // ...
}
```

**Right:**

```java
// The java.nio.file.Path interface is the modern replacement.
// It provides consistent error handling, symbolic link support,
// and platform-independent path manipulation.
Path path = Path.of("data", "output.txt");  // Platform-independent
if (Files.exists(path)) {
    // ...
}
```

**Why it is wrong:** The `java.io.File` class was designed in the 1990s and has numerous design flaws that were fixed by `java.nio.file.Path` in Java 7. The `File` class methods return `boolean` on failure instead of throwing exceptions, making error handling silent and error-prone. `Path` and `Files` provide a cleaner, more consistent API. Use `Path` in all new code.

---

## Key Takeaways

> [!tip] Remember these points
> 1. Java provides two I/O APIs: the legacy `java.io` package (`FileReader`, `FileWriter`, `BufferedReader`) and the modern `java.nio.file` package (`Path`, `Files`). Prefer NIO.2 for new code, but understand the legacy API because you will encounter it in existing codebases.
> 2. **Always use try-with-resources** for file I/O. File handles are operating system resources that must be closed. Leaked file handles cause `Too many open files` errors that crash production servers.
> 3. **Always specify the character encoding explicitly** (preferably `StandardCharsets.UTF_8`). `FileReader` and `FileWriter` use the platform default encoding, which varies between operating systems and causes cross-platform data corruption.
> 4. **Use `BufferedReader` and `BufferedWriter`** for efficient text I/O. The buffer reduces the number of expensive disk I/O operations by reading and writing data in large chunks. For small files, `Files.readString()` and `Files.writeString()` are convenient one-liners.
> 5. **Process large files line by line** using `Files.lines()` (returns a `Stream<String>`) or `BufferedReader.readLine()`. Never load an entire large file into memory with `Files.readString()` or `Files.readAllLines()`, as this causes `OutOfMemoryError`.

---

## Exercise

> [!abstract] Practice before moving to the next note

### Exercise 1: File Read and Write (Easy)

Write a program that:

1. Creates a file called `greeting.txt` and writes three lines of text to it using `BufferedWriter`.
2. Reads the file back using `BufferedReader` and prints each line with a line number prefix.
3. Appends a fourth line to the file using `Files.writeString()` with `StandardOpenOption.APPEND`.
4. Reads the file again and prints all four lines.

> **Hint:** Use `Path.of("greeting.txt")` and try-with-resources for all I/O operations.

### Exercise 2: CSV Processor (Medium)

Create a CSV file called `products.csv` with the following content:

```text
Name,Category,Price,Stock
Laptop,Electronics,85000,15
Mouse,Accessories,1500,50
Keyboard,Accessories,3200,30
Monitor,Electronics,25000,8
Webcam,Accessories,4500,0
```

Write a program that:

1. Reads the CSV file using `Files.lines()` and a Stream pipeline.
2. Skips the header row.
3. Parses each line into a `Product` record.
4. Filters out products with zero stock.
5. Groups the remaining products by category.
6. Calculates the total inventory value per category (price * stock).
7. Prints the results.

> **Hint:** Use `Files.lines()` to get a `Stream<String>`, then `skip(1)` to skip the header, then `map()` to parse each line into a `Product` record. Use `Collectors.groupingBy()` to group by category.

### Exercise 3: Log File Analyzer (Medium)

Write a program that processes a simulated log file. First, create a log file with 100 lines in the format:

```text
[2025-07-10 14:30:45] INFO  - Order #1234 created
[2025-07-10 14:30:46] ERROR - Payment failed for order #1234
[2025-07-10 14:30:47] WARN  - Rate limit approaching for user 5678
```

Then read the log file and:

1. Count the number of ERROR, WARN, and INFO lines.
2. Extract all order numbers mentioned in ERROR lines.
3. Find the earliest and latest timestamps.
4. Write a summary report to a separate file called `log-summary.txt`.

> **Hint:** Use `Files.write()` to generate the log file programmatically. Use `Files.lines()` with stream operations to analyze it. Use `String.split()` or regex to extract fields from each log line.

### Exercise 4: File Copy Utility with Progress (Hard, Optional)

Write a program that copies a large file from one location to another using `BufferedInputStream` and `BufferedOutputStream`. The program should:

1. Read the source file in chunks (e.g., 8KB buffer).
2. Write each chunk to the destination file.
3. Print progress as a percentage after each chunk.
4. Handle the case where the source file does not exist.
5. Verify the copy by comparing file sizes.

Compare the performance of your buffered copy with `Files.copy()` and report the difference.

> **Hint:** Use `InputStream.read(byte[] buffer)` which returns the number of bytes read (or -1 at end of file). Track the total bytes read and divide by the total file size to calculate the percentage.

<details>
<summary>Solution for Exercise 1</summary>

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.io.*;

public class FileReadWrite {
    public static void main(String[] args) throws IOException {
        Path path = Path.of("greeting.txt");

        // Write three lines
        try (BufferedWriter writer = Files.newBufferedWriter(path, StandardCharsets.UTF_8)) {
            writer.write("Hello, Backend World!");
            writer.newLine();
            writer.write("Java File I/O is powerful.");
            writer.newLine();
            writer.write("Always use try-with-resources.");
            writer.newLine();
        }

        // Read and print with line numbers
        System.out.println("--- First Read ---");
        try (BufferedReader reader = Files.newBufferedReader(path, StandardCharsets.UTF_8)) {
            String line;
            int lineNum = 1;
            while ((line = reader.readLine()) != null) {
                System.out.println(lineNum++ + ": " + line);
            }
        }

        // Append a fourth line
        Files.writeString(path, "This line was appended.\n",
            StandardCharsets.UTF_8, StandardOpenOption.APPEND);

        // Read again
        System.out.println("\n--- Second Read ---");
        Files.readAllLines(path, StandardCharsets.UTF_8)
            .forEach(line -> System.out.println("  " + line));
    }
}
```

</details>

<details>
<summary>Solution for Exercise 2</summary>

```java
import java.nio.file.*;
import java.nio.charset.StandardCharsets;
import java.io.IOException;
import java.util.*;
import java.util.stream.*;

record Product(String name, String category, double price, int stock) {}

public class CsvProcessor {
    public static void main(String[] args) throws IOException {
        Path csvPath = Path.of("products.csv");

        // Create the CSV file
        Files.writeString(csvPath,
            "Name,Category,Price,Stock\n" +
            "Laptop,Electronics,85000,15\n" +
            "Mouse,Accessories,1500,50\n" +
            "Keyboard,Accessories,3200,30\n" +
            "Monitor,Electronics,25000,8\n" +
            "Webcam,Accessories,4500,0\n",
            StandardCharsets.UTF_8);

        // Process the CSV
        Map<String, Double> inventoryValueByCategory = Files.lines(csvPath, StandardCharsets.UTF_8)
            .skip(1)  // Skip header
            .filter(line -> !line.isBlank())
            .map(line -> {
                String[] f = line.split(",");
                return new Product(f[0], f[1], Double.parseDouble(f[2]), Integer.parseInt(f[3]));
            })
            .filter(p -> p.stock() > 0)  // Filter out zero stock
            .collect(Collectors.groupingBy(
                Product::category,
                Collectors.summingDouble(p -> p.price() * p.stock())
            ));

        System.out.println("Inventory value by category:");
        inventoryValueByCategory.forEach((cat, value) ->
            System.out.printf("  %-15s: %,.2f BDT%n", cat, value)
        );
    }
}
```

</details>

---

## Related Notes

- [[Java - Basic Input Output - Scanner and System.out]]
- [[Java - Exception Handling - Try Catch Finally Throw Throws]]
- [[Java - Java 8 Streams API]]
- [[Java - Java 8 Lambdas and Functional Interfaces]]

---

## Resources

- [Oracle Java Tutorials: Basic I/O](https://docs.oracle.com/javase/tutorial/essential/io/) - Official documentation covering the entire java.io package.
- [Oracle Java Tutorials: File I/O (NIO.2)](https://docs.oracle.com/javase/tutorial/essential/io/fileio.html) - Official guide to the modern java.nio.file API.
- [Oracle Java Documentation: java.nio.file.Files](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/nio/file/Files.html) - Complete API reference for all file operations.
- [Baeldung: Java Read File](https://www.baeldung.com/java-read-file) - Comprehensive comparison of all file reading methods with performance benchmarks.
- [Baeldung: Java Write File](https://www.baeldung.com/java-write-to-file) - Guide to all file writing approaches.
- [Baeldung: Java NIO vs IO](https://www.baeldung.com/java-nio-vs-io) - Detailed comparison of the legacy and modern I/O APIs.
- [Effective Java by Joshua Bloch - Item 9: Prefer try-with-resources to try-finally](https://www.oreilly.com/library/view/effective-java/9780134686097/) - Why try-with-resources is essential for I/O.
