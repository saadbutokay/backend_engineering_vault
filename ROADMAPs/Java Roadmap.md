# Java Roadmap - Backend Engineering

> [!info] Primary Language
> Java is your primary language. You will spend roughly 70% of your study time here.
> Goal: Junior Backend Engineer -> Mid-Level -> Principal Engineer
---

## Phase 0: Programming Foundations (Weeks 1-4)

> [!tip] Goal
> Understand how programming works. Write basic logic. Get comfortable with syntax.

### Topics
1. [[Java - How Java Works - JVM JRE JDK]]
2. [[Java - Setting Up Environment - IntelliJ and JDK]]
3. [[Java - Variables and Data Types]]
4. [[Java - Operators - Arithmetic Relational Logical]]
5. [[Java - Control Flow - If Else Switch]]
6. [[Java - Loops - For While Do-While]]
7. [[Java - Arrays - 1D and 2D]]
8. [[Java - Strings and String Methods]]
9. [[Java - Methods - Parameters Return Types Overloading]]
10. [[Java - Basic Input Output - Scanner and System.out]]
11. [[Java - Debugging Basics in IntelliJ]]

### Practice Projects
- **Console Calculator**: Takes two numbers and an operator, performs the operation.
- **Number Guessing Game**: Computer picks a random number, user guesses with hints.
- **Grade Calculator**: Takes marks for multiple subjects, calculates GPA.

### Real Backend Connection
> [!note] Why this matters for backend
> Every backend system processes data. Variables store that data. Control flow decides what happens when a user sends a request. Loops process lists of records from a database. You will use these concepts every single day.

---
## Phase 1: Java Core and OOP (Weeks 5-10)

> [!tip] Goal
> Master Object-Oriented Programming. This is the backbone of all Java backend development.

### Topics
1. [[Java - Classes and Objects]]
2. [[Java - Constructors - Default Parameterized Copy]]
3. [[Java - Encapsulation - Getters Setters Access Modifiers]]
4. [[Java - Inheritance - Single Multilevel Hierarchical]]
5. [[Java - Polymorphism - Compile Time and Runtime]]
6. [[Java - Abstraction - Abstract Classes and Interfaces]]
7. [[Java - Static Keyword - Variables Methods Blocks]]
8. [[Java - Final Keyword]]
9. [[Java - this and super Keywords]]
10. [[Java - Exception Handling - Try Catch Finally Throw Throws]]
11. [[Java - Custom Exceptions]]
12. [[Java - Collections Framework Overview]]
13. [[Java - List - ArrayList LinkedList]]
14. [[Java - Set - HashSet TreeSet LinkedHashSet]]
15. [[Java - Map - HashMap TreeMap LinkedHashMap]]
16. [[Java - Queue and Deque]]
17. [[Java - Generics - Classes Methods Wildcards]]
18. [[Java - Comparable and Comparator]]
19. [[Java - File I-O - FileReader FileWriter BufferedReader]]
20. [[Java - Java 8 Lambdas and Functional Interfaces]]
21. [[Java - Java 8 Streams API]]
22. [[Java - Optional Class]]
23. [[Java - Date and Time API - LocalDate LocalDateTime]]
24. [[Java - Enum and Enum Methods]]
25. [[Java - Records - Java 14+]]

### Practice Projects
- **Student Management System (Console)**: Add, view, update, delete students using ArrayList. Use OOP properly with Student class, Course class, etc.
- **Bank Account System**: Account hierarchy (SavingsAccount, CurrentAccount extending Account). Handle transactions, exceptions for insufficient balance.
- **Library Management System**: Books, Members, Borrowing logic. Use HashMap for fast lookups. Use interfaces for Searchable, Borrowable.

### Real Backend Connection
> [!note] Why this matters for backend
> Spring Boot is entirely built on OOP. Every controller, service, and repository is a class. Interfaces define contracts for your services. Exceptions handle errors when a database query fails or a user sends bad data. Collections are how you handle lists of users, orders, or products returned from a database.

---
## Phase 2: Java Advanced and Build Tools (Weeks 11-14)

> [!tip] Goal
> Learn professional Java development practices. Write testable, maintainable code.

### Topics
1. [[Java - Multithreading Basics - Thread Runnable]]
2. [[Java - Thread Synchronization]]
3. [[Java - ExecutorService and Thread Pools]]
4. [[Java - CompletableFuture and Async Programming]]
5. [[Java - Maven - Project Structure POM Dependencies]]
6. [[Java - Gradle Basics]]
7. [[Java - Logging - SLF4J and Logback]]
8. [[Java - Unit Testing - JUnit 5 Basics]]
9. [[Java - Unit Testing - Mockito]]
10. [[Java - Integration Testing Concepts]]
11. [[Java - Design Pattern - Singleton]]
12. [[Java - Design Pattern - Factory]]
13. [[Java - Design Pattern - Builder]]
14. [[Java - Design Pattern - Observer]]
15. [[Java - Design Pattern - Strategy]]
16. [[Java - Design Pattern - Repository Pattern]]
17. [[Java - SOLID Principles]]
18. [[Java - Clean Code Principles]]

### Practice Projects
- **Task Manager CLI with Tests**: A command-line task manager built with Maven. Full JUnit 5 test coverage. Use Builder pattern for Task creation. Use Strategy pattern for different sorting strategies.
- **File Processing Pipeline**: Read a large CSV, process records using thread pools, write output. Use proper logging throughout.

### Real Backend Connection
> [!note] Why this matters for backend
> Every professional Java project uses Maven or Gradle. Every production application has logging. Every serious team requires unit tests. Design patterns appear everywhere in Spring Boot internals. SOLID principles will be asked in interviews.

---
## Phase 3: Databases and SQL (Weeks 15-19)

> [!tip] Goal
> Understand relational databases deeply. Write complex queries. Connect Java to databases.

### Topics
1. [[Database - What is a Database - RDBMS vs NoSQL]]
2. [[Database - PostgreSQL Installation and Setup]]
3. [[Database - SQL Basics - CREATE INSERT UPDATE DELETE]]
4. [[Database - SQL SELECT and Filtering - WHERE LIKE IN BETWEEN]]
5. [[Database - SQL Joins - Inner Left Right Full Cross]]
6. [[Database - SQL Aggregation - GROUP BY HAVING COUNT SUM AVG]]
7. [[Database - SQL Subqueries and CTEs]]
8. [[Database - SQL Indexes - Types and When to Use]]
9. [[Database - SQL Transactions - ACID Properties]]
10. [[Database - Database Normalization - 1NF 2NF 3NF BCNF]]
11. [[Database - Database Design - ER Diagrams]]
12. [[Database - Constraints - Primary Key Foreign Key Unique Check]]
13. [[Java - JDBC - Connection Statement ResultSet]]
14. [[Java - JDBC - PreparedStatement and SQL Injection]]
15. [[Java - JDBC - Connection Pooling with HikariCP]]
16. [[Java - JDBC - DAO Pattern Implementation]]

### Practice Projects
- **Employee Database CRUD**: Full JDBC application. Create tables, insert employees, search by department, update salaries, delete records. Use PreparedStatement everywhere. Use HikariCP for connection pooling.
- **E-Commerce Database Design**: Design tables for Users, Products, Orders, OrderItems, Payments. Write complex queries: top 5 customers by spending, monthly revenue report, products never ordered.

### Real Backend Connection
> [!note] Why this matters for backend
> The database is the heart of every backend system. You will write SQL queries daily. Understanding joins, indexes, and transactions separates junior developers from mid-level ones. Every Spring Boot application connects to a database.

---
## Phase 4: Spring Boot Fundamentals (Weeks 20-28)

> [!tip] Goal
> Build production-grade REST APIs. This is the most important phase for getting hired.

### Topics
1. [[Spring - What is Spring Framework - IoC and DI]]
2. [[Spring - Spring Boot vs Spring - Auto Configuration]]
3. [[Spring - Project Setup - Spring Initializr]]
4. [[Spring - Application Properties and Profiles]]
5. [[Spring - REST Concepts - HTTP Methods Status Codes]]
6. [[Spring - Controllers - @RestController @GetMapping @PostMapping]]
7. [[Spring - Request Handling - @PathVariable @RequestParam @RequestBody]]
8. [[Spring - Service Layer - @Service Business Logic]]
9. [[Spring - Repository Layer - @Repository]]
10. [[Spring - Spring Data JPA - CrudRepository JpaRepository]]
11. [[Spring - Entity Mapping - @Entity @Table @Column @Id]]
12. [[Spring - Relationships - @OneToMany @ManyToOne @ManyToMany]]
13. [[Spring - DTO Pattern - Request and Response DTOs]]
14. [[Spring - MapStruct - Object Mapping]]
15. [[Spring - Validation - @Valid @NotNull @Size Custom Validators]]
16. [[Spring - Global Exception Handling - @ControllerAdvice]]
17. [[Spring - Pagination and Sorting]]
18. [[Spring - Spring Boot Testing - @SpringBootTest @WebMvcTest]]
19. [[Spring - H2 Database for Development]]
20. [[Spring - Connecting to PostgreSQL in Production]]

### Practice Projects (Portfolio Worthy)
- **Blog REST API**: Full CRUD for Posts, Comments, Users. JPA relationships. Pagination. Validation. Exception handling. DTOs. This is your first portfolio project.
- **Task Management API**: Users can create projects, add tasks, assign tasks, mark complete. JWT-ready structure. Proper layered architecture.

### Real Backend Connection
> [!note] Why this matters for backend
> This IS backend development in Java. 90% of Java backend jobs in Bangladesh and remotely require Spring Boot. Every API you build at work will look like what you learn here. The layered architecture (Controller -> Service -> Repository) is the industry standard.

---
## Phase 5: Spring Boot Advanced (Weeks 29-36)

> [!tip] Goal
> Build secure, scalable, production-ready applications. This makes you hireable at good companies.

### Topics
1. [[Spring - Spring Security Fundamentals]]
2. [[Spring - JWT Authentication - Access and Refresh Tokens]]
3. [[Spring - Role Based Authorization - @PreAuthorize]]
4. [[Spring - OAuth2 Login - Google GitHub]]
5. [[Spring - Caching - @Cacheable with Redis]]
6. [[Spring - Redis Setup and Integration]]
7. [[Spring - Messaging - RabbitMQ Basics]]
8. [[Spring - Messaging - Apache Kafka Basics]]
9. [[Spring - Async Processing - @Async]]
10. [[Spring - Scheduling - @Scheduled Cron Jobs]]
11. [[Spring - File Upload and Download]]
12. [[Spring - API Documentation - Swagger OpenAPI 3]]
13. [[Spring - Actuator - Health Checks and Metrics]]
14. [[Spring - Rate Limiting]]
15. [[Spring - WebSockets Basics]]
16. [[Spring - Internationalization - i18n]]

### Practice Projects (Portfolio Worthy)
- **E-Commerce Backend API**: Users, Products, Categories, Cart, Orders, Payments (mock). JWT auth with roles (ADMIN, CUSTOMER). Redis caching for product listings. Swagger documentation. This is your main portfolio project.
- **Real-Time Chat Backend**: WebSocket-based chat with Spring Boot. User authentication. Message persistence. Room/channel support.

### Real Backend Connection
> [!note] Why this matters for backend
> Security is non-negotiable in production. Every real application has authentication and authorization. Caching makes your API fast. Message queues handle background processing like sending emails or processing payments. These skills move you from junior to mid-level.

---
## Phase 6: DevOps and Deployment (Weeks 37-41)

> [!tip] Goal
> Deploy your applications. Understand the infrastructure around your code.

### Topics
1. [[DevOps - Git Fundamentals - Branching Merging Rebasing]]
2. [[DevOps - Git Workflow - Feature Branch Strategy]]
3. [[DevOps - Linux Command Line Essentials]]
4. [[DevOps - Docker Fundamentals - Images Containers Dockerfile]]
5. [[DevOps - Docker Compose - Multi Container Setup]]
6. [[DevOps - Dockerizing Spring Boot Application]]
7. [[DevOps - CI CD Concepts - GitHub Actions]]
8. [[DevOps - GitHub Actions Pipeline for Java]]
9. [[DevOps - Cloud Basics - AWS EC2 RDS S3]]
10. [[DevOps - Deploying to Railway or Render]]
11. [[DevOps - Nginx as Reverse Proxy]]
12. [[DevOps - Environment Variables and Secrets Management]]
13. [[DevOps - SSL and HTTPS Basics]]

### Practice Projects
- **Full Deployment Pipeline**: Take your E-Commerce API. Write a Dockerfile. Create a docker-compose.yml with PostgreSQL and Redis. Set up GitHub Actions to build and test on every push. Deploy to Railway or a free AWS tier.

### Real Backend Connection
> [!note] Why this matters for backend
> Code that only runs on your laptop is useless. Every company expects you to understand Docker and basic CI/CD. Knowing how to deploy your own application makes you stand out massively in the Bangladesh job market where many juniors cannot do this.

---

## Phase 7: System Design and Architecture (Weeks 42-48)
> [!tip] Goal
> Think like a mid-level engineer. Design systems that scale.

### Topics
1. [[System Design - Monolith vs Microservices]]
2. [[System Design - API Gateway Pattern]]
3. [[System Design - Service Discovery]]
4. [[System Design - Load Balancing]]
5. [[System Design - Database Scaling - Read Replicas Sharding]]
6. [[System Design - CAP Theorem]]
7. [[System Design - Caching Strategies - Write Through Write Back]]
8. [[System Design - Message Queue Architecture]]
9. [[System Design - Event Driven Architecture]]
10. [[System Design - Rate Limiting Algorithms]]
11. [[System Design - CDN and Static Content]]
12. [[System Design - Designing URL Shortener]]
13. [[System Design - Designing Chat System]]
14. [[System Design - Designing E-Commerce Order System]]

### Practice Projects
- **Microservices Order System**: Break your E-Commerce monolith into Order Service, Product Service, User Service. Use RabbitMQ or Kafka for inter-service communication. API Gateway with Spring Cloud Gateway.

### Real Backend Connection
> [!note] Why this matters for backend
> System design interviews are standard for mid-level positions and above. Understanding architecture helps you make better decisions even as a junior. This knowledge separates code writers from engineers.

---
## Phase 8: Data Engineering with Java (Weeks 49-56)

> [!tip] Goal
> Explore your interest in data engineering. Build data pipelines.

### Topics
1. [[Data Engineering - What is Data Engineering - ETL ELT]]
2. [[Data Engineering - Apache Spark with Java - Basics]]
3. [[Data Engineering - Spark DataFrames and Datasets]]
4. [[Data Engineering - Apache Kafka In Depth]]
5. [[Data Engineering - Kafka Producers and Consumers in Java]]
6. [[Data Engineering - Data Warehousing Concepts]]
7. [[Data Engineering - Apache Airflow Basics]]
8. [[Data Engineering - Batch vs Stream Processing]]
9. [[Data Engineering - Data Lake vs Data Warehouse]]
10. [[Data Engineering - Apache Flink Introduction]]

### Practice Projects
- **Real-Time Data Pipeline**: Kafka producer generates simulated e-commerce events (clicks, purchases). Spark Streaming consumer processes events in real time. Aggregated results stored in PostgreSQL. Airflow orchestrates daily batch reports.

### Real Backend Connection
> [!note] Why this matters for backend
> Data engineering is one of the highest-paying specializations. Many backend engineers transition into data engineering. Companies like Pathao, ShopUp, and bKash in Bangladesh have growing data teams. Remote data engineering roles pay very well.

---

## Phase 9: Principal Level Mastery (Ongoing)
> [!tip] Goal
> Long-term growth. Architecture leadership. Technical decision-making.

### Topics
1. [[Advanced - Domain Driven Design - Entities Value Objects Aggregates]]
2. [[Advanced - Event Sourcing]]
3. [[Advanced - CQRS Pattern]]
4. [[Advanced - Distributed Transactions - Saga Pattern]]
5. [[Advanced - Performance Tuning - JVM Profiling]]
6. [[Advanced - Garbage Collection Tuning]]
7. [[Advanced - Java Memory Model]]
8. [[Advanced - Reactive Programming - Project Reactor WebFlux]]
9. [[Advanced - GraalVM and Native Images]]
10. [[Advanced - Architecture Decision Records]]
11. [[Advanced - Technical Leadership and Mentoring]]
12. [[Advanced - Observability - OpenTelemetry Prometheus Grafana]]

### Practice Projects
- **High-Performance Event Sourced System**: Build a banking ledger using event sourcing and CQRS. Use WebFlux for reactive endpoints. Profile and optimize JVM performance. Full observability stack.

---
## Job Readiness Checklist

> [!warning] Before applying for junior backend roles, make sure you have:

- [ ] Completed Phases 0 through 5
- [ ] At least 2 portfolio projects deployed and live
- [ ] GitHub profile with clean, documented repositories
- [ ] Basic understanding of Docker and deployment (Phase 6)
- [ ] A resume highlighting your projects and skills
- [ ] Practiced SQL queries on LeetCode or HackerRank
- [ ] Practiced Java coding problems on LeetCode (Easy and Medium)
- [ ] Understanding of REST API design principles

---
## Recommended Resources

| Resource | Type | Use For |
|----------|------|---------|
| MOOC.fi Java Programming | Free Course | Phase 0 and 1 |
| Spring Boot official docs | Documentation | Phase 4 and 5 |
| Baeldung.com | Articles | All Spring phases |
| LeetCode | Practice | Coding interviews |
| System Design Primer (GitHub) | Free Guide | Phase 7 |
| Docker official docs | Documentation | Phase 6 |
| PostgreSQL Tutorial | Free Guide | Phase 3 |

---