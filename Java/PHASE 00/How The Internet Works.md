## Overview

Every backend system you build will communicate over a network. Your Java application will receive HTTP requests from clients, talk to databases on other machines, publish messages to brokers, and call external APIs. None of this works without understanding the layers beneath. This section covers the foundational networking concepts you need before writing a single line of server code.

---

## Core Concepts

### The Client-Server Model

The internet operates on a client-server architecture:

- **Client** — The entity that initiates a request. This is a web browser, a mobile app, another backend service, or a CLI tool like `curl`.
- **Server** — The entity that listens for requests, processes them, and returns responses. Your Java Spring Boot application will be a server.

A single machine can act as both client and server simultaneously. When your backend calls a payment gateway API, your server becomes a client to that external service.

### IP Addresses

Every device on a network has an **IP address** — a numerical label used to identify it.

- **IPv4** — 32-bit, written as four decimal octets: `192.168.1.1`. Approximately 4.3 billion addresses (exhausted).
- **IPv6** — 128-bit, written in hexadecimal: `2001:0db8:85a3::8a2e:0370:7334`. Effectively unlimited addresses.

Key distinctions:

- **Public IP** — Routable on the internet. Assigned by your ISP.
- **Private IP** — Used within local networks (e.g., `192.168.x.x`, `10.x.x.x`, `172.16-31.x.x`). Not routable on the public internet.
- **Loopback** — `127.0.0.1` (IPv4) or `::1` (IPv6). Refers to the machine itself. When you run a Spring Boot app locally and access `localhost:8080`, you are using the loopback address.

### Ports

An IP address identifies a machine. A **port** identifies a specific process or service on that machine.

- Ports range from 0 to 65535.
- **Well-known ports (0-1023):**
    - 22 — SSH
    - 80 — HTTP
    - 443 — HTTPS
    - 5432 — PostgreSQL
    - 6379 — Redis
    - 9092 — Kafka
- **Registered ports (1024-49151):** Assigned to specific applications.
- **Dynamic/ephemeral ports (49152-65535):** Used temporarily by clients.

When your Spring Boot app starts on port 8080, it is listening for incoming TCP connections on that port. A request to `192.168.1.10:8080/api/users` targets the machine at that IP, the process on port 8080, and the specific resource path `/api/users`.

### DNS (Domain Name System)

Humans use domain names (`api.stripe.com`). Machines use IP addresses (`104.16.24.10`). DNS translates between them.

Resolution process (simplified):

1. Your machine checks its local DNS cache.
2. If not cached, it queries a **recursive resolver** (usually your ISP or a public resolver like `8.8.8.8`).
3. The resolver queries the **root nameservers** (13 globally).
4. The root directs to the **TLD nameserver** (`.com`, `.org`, `.io`).
5. The TLD directs to the **authoritative nameserver** for the domain.
6. The authoritative server returns the IP address.
7. The result is cached at every level for a duration specified by the **TTL (Time to Live)**.

As a backend engineer, you will configure DNS records for your services, understand DNS propagation delays during deployments, and debug DNS resolution failures in containerized environments.

### The TCP/IP Stack

Network communication is organized into layers. The most common model is the **TCP/IP model** (4 layers), though the **OSI model** (7 layers) is also referenced.

**TCP/IP Model:**

```
Application Layer    HTTP, HTTPS, DNS, SMTP, FTP, SSH
Transport Layer      TCP, UDP
Internet Layer       IP, ICMP
Network Access       Ethernet, Wi-Fi
```

**What each layer does:**

- **Network Access** — Physical transmission of bits over cables or radio waves. You will rarely interact with this.
- **Internet Layer (IP)** — Addresses and routes packets across networks. IP is connectionless and unreliable (packets can be lost, duplicated, or reordered).
- **Transport Layer (TCP/UDP)** — Provides end-to-end communication between processes.
    - **TCP (Transmission Control Protocol)** — Connection-oriented, reliable, ordered delivery. Uses a three-way handshake (SYN, SYN-ACK, ACK). Retransmits lost packets. This is what HTTP uses.
    - **UDP (User Datagram Protocol)** — Connectionless, unreliable, fast. No handshake, no retransmission. Used for DNS queries, video streaming, gaming.
- **Application Layer** — The protocols your applications speak directly. HTTP, HTTPS, SMTP, etc.

### HTTP and HTTPS

**HTTP (HyperText Transfer Protocol)** is the foundation of web communication. It is a request-response protocol built on top of TCP.

**HTTP Request Structure:**

```
GET /api/users?page=1&limit=10 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
Accept: application/json
Content-Type: application/json

{"name": "John"}
```

Components:
- **Method** — `GET`, `POST`, `PUT`, `PATCH`, `DELETE` (more in Phase 05)
- **URL** — The resource being requested
- **HTTP Version** — `HTTP/1.1`, `HTTP/2`, `HTTP/3`
- **Headers** — Key-value metadata pairs
- **Body** — The payload (present in POST, PUT, PATCH; typically absent in GET)

**HTTP Response Structure:**

```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-cache

{"id": 1, "name": "John", "email": "john@example.com"}
```

Components:
- **Status Line** — HTTP version, status code, reason phrase
- **Headers** — Metadata about the response
- **Body** — The response payload

**HTTPS** is HTTP encrypted with **TLS (Transport Layer Security)**. The TLS handshake establishes an encrypted channel before any HTTP data is transmitted. This prevents eavesdropping and tampering. In production, every API you build must use HTTPS. Spring Boot makes this straightforward with configuration.

**HTTP Versions:**

- **HTTP/1.1** — Text-based, one request per connection (or pipelining with limitations). Still widely used.
- **HTTP/2** — Binary, multiplexed (multiple requests over one connection), header compression, server push. Most modern browsers and servers use this.
- **HTTP/3** — Built on QUIC (UDP-based), reduces latency, handles connection migration. Growing adoption.

### What a URL Is

A **URL (Uniform Resource Locator)** identifies a resource on the internet.

```
https://api.example.com:8443/v1/users/42?fields=name,email#section
│       │               │    │        │  │               │
│       │               │    │        │  │               └─ Fragment (client-side only)
│       │               │    │        │  └─ Query string (key-value parameters)
│       │               │    │        └─ Path (resource identifier)
│       │               │    └─ Port (optional, defaults to 80/443)
│       │               └─ Host (domain name or IP)
│       └─ Scheme (protocol)
└─ Full URL
```

In your Spring Boot controllers, you will map paths like `/v1/users/{id}` and extract query parameters like `?fields=name,email`.

### What an API Is

An **API (Application Programming Interface)** is a contract between two software systems defining how they communicate.

In backend engineering, when we say "API," we almost always mean a **web API** — an interface exposed over HTTP that accepts requests and returns responses, typically in JSON format.

Types of APIs you will encounter:

- **REST (Representational State Transfer)** — Resource-oriented, uses HTTP methods. The most common. You will build REST APIs in Phase 05.
- **GraphQL** — Client specifies exactly what data it needs. Single endpoint. Covered as an overview.
- **gRPC** — High-performance, binary protocol using Protocol Buffers. Common in microservice-to-microservice communication. Covered in Phase 06.
- **SOAP** — XML-based, legacy enterprise protocol. You may encounter it in older fintech systems. Awareness only.

### JSON (JavaScript Object Notation)

JSON is the dominant data interchange format for web APIs. It is human-readable and language-independent.

```json
{
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "active": true,
    "balance": 1500.75,
    "roles": ["user", "admin"],
    "address": {
        "street": "123 Main St",
        "city": "New York"
    },
    "metadata": null
}
```

JSON data types: string, number, boolean, null, object (`{}`), array (`[]`).

In Java, you will serialize Java objects to JSON and deserialize JSON to Java objects using the **Jackson** library (the default in Spring Boot). This is covered in Phase 01 and used extensively from Phase 05 onward.

### Latency, Bandwidth, and Throughput

- **Latency** — The time it takes for a single request to travel from client to server and back. Measured in milliseconds (ms). A round-trip from New York to London is approximately 70 ms. Within a data center, it is under 1 ms.
- **Bandwidth** — The maximum data transfer rate of a network connection. Measured in bits per second (Mbps, Gbps). Think of it as the width of a pipe.
- **Throughput** — The actual rate of successful data transfer. Measured in requests per second (RPS) or bytes per second. Think of it as how much water actually flows through the pipe.

A system can have high bandwidth but high latency (satellite internet) or low bandwidth but low latency (local network). Backend engineers optimize for both, but latency is usually the primary concern for user-facing APIs.

---

## Code Examples

**Making an HTTP request from the terminal:**

```bash
# GET request
curl -X GET https://jsonplaceholder.typicode.com/users/1

# Response:
# {
#   "id": 1,
#   "name": "Leanne Graham",
#   "username": "Bret",
#   "email": "Sincere@april.biz",
#   ...
# }
```

**Making a POST request with JSON:**

```bash
curl -X POST https://jsonplaceholder.typicode.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title": "Test", "body": "Hello", "userId": 1}'
```

**Checking DNS resolution:**

```bash
nslookup api.stripe.com
# Server:  8.8.8.8
# Address: 8.8.8.8#53
#
# Name: api.stripe.com
# Address: 104.16.24.10

dig api.stripe.com +short
# 104.16.24.10
```

**Checking which ports are open on your machine:**

```bash
# macOS
lsof -i -P | grep LISTEN

# Linux
ss -tlnp
```

**A minimal Java HTTP client (Java 11+ HttpClient):**

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class HttpClientExample {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://jsonplaceholder.typicode.com/users/1"))
            .header("Accept", "application/json")
            .GET()
            .build();

        HttpResponse<String> response = client.send(
            request, HttpResponse.BodyHandlers.ofString()
        );

        System.out.println("Status: " + response.statusCode());
        System.out.println("Body: " + response.body());
    }
}
```

This is the Java standard library HTTP client. In production Spring Boot applications, you will more commonly use `RestClient`, `WebClient`, or `OpenFeign`, but understanding the underlying mechanics matters.

---

## Important Notes

- HTTP is stateless. Each request is independent. The server does not remember previous requests unless you implement session management or token-based authentication. This is a fundamental design constraint you will work within throughout your career.
- HTTPS is not optional in production. Browsers flag HTTP as insecure. Payment processors (Stripe, Adyen) require HTTPS. PCI DSS compliance mandates encryption in transit.
- DNS resolution adds latency. A cold DNS lookup can take 20-100 ms. This is why connection pooling and DNS caching matter in high-throughput backend systems.
- TCP's three-way handshake adds one round-trip of latency before any data is sent. TLS adds one or two more round-trips. This is why connection reuse (keep-alive, connection pooling) is critical for performance.
- The distinction between TCP and UDP matters when you study Kafka (which uses TCP) and when you encounter real-time streaming systems that may use UDP-based protocols.
- JSON is not the only data format. XML is still used in SOAP APIs and some banking systems. Protocol Buffers are used in gRPC. Avro is used in Kafka. You will encounter all of these in fintech.

---

## Practice

1. Open your terminal. Run `curl -v https://httpbin.org/get` and study the verbose output. Identify the DNS lookup, TCP handshake, TLS handshake, HTTP request, and HTTP response.
2. Run `dig google.com` and identify the TTL value. What does it mean for caching?
3. Run `curl -X POST https://httpbin.org/post -H "Content-Type: application/json" -d '{"key":"value"}'` and examine the response.
4. Write the Java `HttpClientExample` above, compile it, and run it. Verify the output.
5. In your own words, explain the difference between latency and throughput. Add this to your glossary.

---

## References

- MDN Web Docs — HTTP: https://developer.mozilla.org/en-US/docs/Web/HTTP
- MDN Web Docs — How the Web Works: https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/How_the_Web_works
- Java HttpClient Documentation: https://docs.oracle.com/en/java/javase/21/docs/api/java.net.http/java/net/http/HttpClient.html
- RFC 2616 (HTTP/1.1): https://datatracker.ietf.org/doc/html/rfc2616
- RFC 9110 (HTTP Semantics, current): https://datatracker.ietf.org/doc/html/rfc9110
