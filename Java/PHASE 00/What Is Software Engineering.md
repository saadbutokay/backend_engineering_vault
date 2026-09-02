## Overview

Programming is writing code. Software engineering is designing, building, testing, deploying, and maintaining systems that solve real problems for real users under real constraints. The difference matters because anyone can write a script that works on their laptop. Engineering is making that script run reliably at 3 AM on a Saturday processing ten thousand transactions per second without losing a single cent. This section defines the discipline you are entering, the roles within it, and the daily reality of the job you are training for.

---

## Core Concepts

### Software Engineering vs Programming

**Programming** is the act of writing instructions a computer can execute. It is a skill, like typing or arithmetic.

**Software engineering** is the application of engineering principles to software development. It includes:

- Understanding requirements (what problem are we solving and for whom)
- Designing systems (how will the components interact)
- Writing code (the programming part)
- Testing (verifying correctness under expected and unexpected conditions)
- Deploying (getting the software to users)
- Monitoring (observing behavior in production)
- Maintaining (fixing bugs, adding features, updating dependencies)
- Collaborating (working with other engineers, product managers, designers)

A programmer writes a function that calculates interest. A software engineer writes a function that calculates interest correctly across 47 currencies, handles leap years, survives a database outage mid-calculation, logs every computation for audit compliance, and can be modified by a new team member six months from now without breaking anything.

### Software Development Life Cycle (SDLC)

The SDLC is a framework describing the stages a software project passes through from conception to retirement. Several models exist. The most common in modern teams:

**Waterfall (legacy, rare in new projects):**
```
Requirements → Design → Implementation → Testing → Deployment → Maintenance
```

Each phase completes before the next begins. Rigid. Changes are expensive. Still used in some government and defense contracts.

**Iterative:**
```
Requirements → Design → Implement → Test → Review → (repeat with refinements)
```

Build a rough version, get feedback, improve. More flexible than waterfall.

**Agile (dominant in modern teams):**
```
Continuous cycles of: Plan → Build → Test → Review → Adapt
```

Work is divided into short iterations (1-4 weeks). Requirements evolve based on feedback. The team adapts continuously. This is what you will encounter in nearly every tech company and most fintech firms.

### Agile, Scrum, and Kanban
**Agile** is a philosophy, not a process. It is defined by the Agile Manifesto (2001):
- Individuals and interactions over processes and tools
- Working software over comprehensive documentation
- Customer collaboration over contract negotiation
- Responding to change over following a plan

**Scrum** is the most common Agile framework:
- **Sprint** — A fixed timebox (usually 2 weeks) during which the team delivers a potentially shippable increment.
- **Product Backlog** — A prioritized list of all desired features, fixes, and improvements.
- **Sprint Backlog** — The subset of backlog items the team commits to completing in the current sprint.
- **Daily Standup** — A 15-minute meeting where each team member answers: What did I do yesterday? What will I do today? Any blockers?
- **Sprint Planning** — The team selects backlog items for the upcoming sprint and breaks them into tasks.
- **Sprint Review** — The team demonstrates completed work to stakeholders.
- **Sprint Retrospective** — The team reflects on what went well and what to improve.
- **Roles:**
    - **Product Owner** — Defines priorities, represents the customer.
    - **Scrum Master** — Facilitates the process, removes blockers.
    - **Development Team** — Builds the software (that is you).

**Kanban** is a flow-based method:
- No fixed sprints. Work flows continuously.
- A **Kanban board** visualizes work in columns: To Do → In Progress → In Review → Done.
- **WIP limits** (Work in Progress) restrict how many items can be in each column simultaneously. This prevents context switching and bottlenecks.
- Focus on cycle time (how long an item takes from start to finish).
- Common in operations, support, and maintenance teams.

Most teams you join will use Scrum or a hybrid of Scrum and Kanban ("Scrumban").

### Version Control
**Version control** is a system that records changes to files over time so you can recall specific versions later.

**Why it matters:**
- **History** — Every change is recorded with who made it, when, and why. You can trace a bug to the exact commit that introduced it.
- **Collaboration** — Multiple engineers work on the same codebase simultaneously without overwriting each other's work.
- **Branching** — You can create an isolated copy of the code to experiment, build a feature, or fix a bug without affecting the main codebase.
- **Rollback** — If a deployment breaks production, you can revert to the last known good version in minutes.
- **Code review** — Changes are submitted as pull requests, reviewed by peers, and merged only after approval.

**Git** is the dominant version control system. **GitHub**, **GitLab**, and **Bitbucket** are platforms that host Git repositories and add collaboration features (pull requests, issue tracking, CI/CD).
You will learn Git in depth in Phase 02. For now, understand that every line of production code you write will pass through Git. There are no exceptions.

### Testing
**Testing** is the process of verifying that software behaves as expected.

**Why it matters:**
- Untested code is broken code you have not discovered yet.
- In fintech, a bug in a payment calculation can cost millions and trigger regulatory action.
- Tests give you confidence to refactor and deploy without fear.
- Tests serve as living documentation of expected behavior.

**Types of tests (from smallest scope to largest):**

| Type | Scope | Speed | Example |
|------|-------|-------|---------|
| Unit | Single function or class | Milliseconds | Testing that `calculateInterest(1000, 0.05, 1)` returns `50.00` |
| Integration | Multiple components together | Seconds | Testing that the payment service correctly writes to the database |
| End-to-End | Full user workflow | Minutes | Testing that a user can register, add a card, and make a payment |
| Performance | System under load | Minutes to hours | Testing that the API handles 10,000 requests per second |
| Security | Vulnerability scanning | Varies | Testing for SQL injection, XSS, broken authentication |

**Testing is not optional.** In professional Java development, you will write tests using JUnit 5 and Mockito. Test coverage below 80% is generally considered unacceptable in fintech. You will learn testing in Phase 01 and practice it in every subsequent phase.

### Deployment
**Deployment** is the process of moving software from a development environment to a production environment where real users interact with it.

**Environments (typical progression):**

```
Local (your laptop)
    → Development (shared dev server)
        → Staging (mirrors production, for final testing)
            → Production (live, real users, real money)
```

**Deployment strategies:**
- **Rolling update** — Replace instances gradually. Zero downtime. Most common.
- **Blue-green** — Two identical environments. Switch traffic from old (blue) to new (green). Instant rollback.
- **Canary** — Release to a small percentage of users first. Monitor for errors. Gradually increase traffic.
- **Feature flags** — Deploy code to production but hide new features behind toggles. Enable gradually.

**CI/CD (Continuous Integration / Continuous Deployment):**
- **CI** — Every code change triggers automated builds and tests. If tests fail, the change is rejected.
- **CD** — If tests pass, the change is automatically deployed to staging or production.

You will learn Docker, CI/CD pipelines, and cloud deployment in Phase 07. For now, understand that modern teams deploy multiple times per day, not once per quarter.

### Engineering Roles

| Role | Focus | Key Technologies |
|------|-------|-----------------|
| **Frontend Engineer** | User interfaces, browser-side logic | React, Angular, TypeScript, CSS |
| **Backend Engineer** | Server-side logic, APIs, databases, business rules | Java, Spring Boot, PostgreSQL, Kafka, Redis |
| **Fullstack Engineer** | Both frontend and backend | Combination of above |
| **DevOps Engineer** | Infrastructure, CI/CD, deployment, monitoring | Docker, Kubernetes, Terraform, AWS, Jenkins |
| **Site Reliability Engineer (SRE)** | System reliability, uptime, incident response | Monitoring, alerting, capacity planning, chaos engineering |
| **Data Engineer** | Data pipelines, ETL, data warehouses | Spark, Airflow, Kafka, Snowflake, dbt |
| **Machine Learning Engineer** | Model training, serving, MLOps | Python, TensorFlow, PyTorch, MLflow |
| **Security Engineer** | Application and infrastructure security | Penetration testing, OWASP, encryption, IAM |
| **QA Engineer** | Test strategy, automation, quality assurance | Selenium, Cypress, JUnit, performance testing |
| **Platform Engineer** | Internal developer tools and infrastructure | Kubernetes, service mesh, internal frameworks |

### What a Backend Engineer Does Day-to-Day
This is the role you are training for. A typical day:

**Morning:**
- Check monitoring dashboards (Grafana, Datadog, CloudWatch) for overnight errors or anomalies.
- Read pull requests from teammates. Leave comments, approve or request changes.
- Attend the daily standup (15 minutes). Report progress and blockers.

**Core work (bulk of the day):**
- Design and implement a new API endpoint. Write the controller, service layer, repository, and database migration.
- Write unit and integration tests for the new code.
- Debug a production issue. Trace a failed transaction through logs, identify the root cause, write a fix, write a test that reproduces the bug, submit a pull request.
- Optimize a slow database query. Run `EXPLAIN ANALYZE`, add an index, verify the improvement.
- Review the architecture for an upcoming feature. Write a design document. Discuss tradeoffs with the team.
- Pair program with a junior engineer on a complex problem.

**Afternoon:**
- Attend a sprint planning or refinement meeting. Estimate effort for upcoming stories.
- Respond to questions from frontend engineers about API contracts.
- Update documentation for a service you own.
- Deploy your changes through the CI/CD pipeline. Monitor the rollout.

**Ongoing responsibilities:**
- On-call rotation (typically one week every 6-8 weeks). You carry a pager and respond to production incidents at any hour.
- Mentoring newer engineers.
- Evaluating new tools and libraries.
- Participating in post-mortems after incidents.

### Why Java Dominates Enterprise and Fintech Backend
Java is the primary language for this roadmap because of its specific strengths in the domains you are targeting:

**Type safety and correctness:**
- Java's static type system catches errors at compile time, not at runtime. In a payment system, a type error that slips to production can move real money to the wrong account. Java's compiler prevents entire categories of bugs that dynamic languages allow.

**Concurrency and performance:**
- Java's threading model, concurrent collections, and (as of Java 21) virtual threads make it possible to handle hundreds of thousands of concurrent connections efficiently. Fintech systems process millions of transactions per day. Python's Global Interpreter Lock (GIL) makes this significantly harder.

**Mature ecosystem:**
- Spring Boot, Spring Security, Spring Data, Hibernate, Kafka clients, gRPC — the Java ecosystem has battle-tested libraries for every backend concern. These libraries have been hardened over decades of production use in banks and trading firms.

**Long-term stability:**
- Java has strict backward compatibility guarantees. Code written for Java 8 in 2014 still compiles and runs on Java 21 in 2024. Fintech systems have lifespans of 10-20 years. This stability matters enormously.

**Regulatory compliance:**
- Java's strong typing, comprehensive logging, and mature security frameworks (Spring Security, Bouncy Castle) make it easier to meet PCI DSS, SOC 2, and other regulatory requirements that fintech companies must satisfy.

**Talent pool and institutional knowledge:**
- Most major banks (JPMorgan, Goldman Sachs, Morgan Stanley, Barclays) and fintech companies (Stripe, Square, Adyen, Revolut) have massive Java codebases. The institutional knowledge, tooling, and hiring pipelines are built around Java.

**JVM ecosystem:**
- The JVM runs not just Java but also Kotlin, Scala, and Clojure. Learning the JVM gives you access to all of these languages without changing your deployment infrastructure.

### Reading Documentation as a Skill
Documentation is the primary source of truth for every library, framework, and API you will use. Learning to read it efficiently is a force multiplier.

**Types of documentation you will encounter:**

- **Javadoc** — Java's built-in documentation format. Generated from source code comments. Example: https://docs.oracle.com/en/java/javase/21/docs/api/
- **Official framework docs** — Spring Boot reference: https://docs.spring.io/spring-boot/docs/current/reference/html/
- **API reference** — Lists every class, method, parameter, and return type. Dense but precise.
- **Guides and tutorials** — Narrative explanations with examples. Good for learning, bad for reference.
- **README files** — Project overview, setup instructions, quick start. Found on GitHub.
- **RFCs and JEPs** — Design documents for language and protocol changes. JDK Enhancement Proposals: https://openjdk.org/jeps/0

**How to read documentation effectively:**

1. **Start with the overview or quick start.** Understand what the tool does and its core concepts before diving into details.
2. **Jump to the API reference when you need specifics.** Do not read it cover to cover. Search for the class or method you need.
3. **Read the method signature carefully.** Note the parameter types, return type, and declared exceptions. In Java, the type signature tells you most of what you need to know.
4. **Check the "Since" tag.** It tells you which Java or library version introduced the feature. If you are on Java 17 and a method says "Since 21," it will not compile.
5. **Look at the examples.** Good documentation includes code snippets. Study them, then adapt them to your use case.
6. **Read the caveats and warnings.** These are usually in "Note" or "Warning" blocks. They describe edge cases and common mistakes.
7. **Check the changelog or release notes** when upgrading versions. Breaking changes are listed here.

### How to Ask Technical Questions

You will get stuck. Frequently. The quality of your questions determines the speed of your unblocking.

**The format for a good technical question:**

1. **State the goal.** What are you trying to achieve?
2. **State the problem.** What is happening instead?
3. **Provide context.** Language version, framework version, operating system, relevant configuration.
4. **Show what you tried.** List the approaches you already attempted and why they did not work.
5. **Include the exact error message.** Copy and paste the full stack trace, not a paraphrase.
6. **Provide a minimal reproducible example.** The smallest piece of code that demonstrates the problem.

**Bad question:**
> "My Spring Boot app is not working. Help."

**Good question:**
> "I am building a Spring Boot 3.2 application with Java 21 on macOS 14. I am trying to connect to a PostgreSQL 16 database using Spring Data JPA. When the application starts, I get the following error:
>
> `org.postgresql.util.PSQLException: Connection to localhost:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.`
>
> I have verified that PostgreSQL is running (`pg_isready` returns `accepting connections`). I can connect via `psql -U postgres -d mydb` from the terminal. My `application.yml` has:
>
> ```yaml
> spring:
>   datasource:
>     url: jdbc:postgresql://localhost:5432/mydb
>     username: postgres
>     password: secret
> ```
>
> I tried changing `localhost` to `127.0.0.1` but got the same error. I also checked `postgresql.conf` and `listen_addresses` is set to `'*'`. What else could cause this?"

The good question gives the reader everything they need to diagnose the problem without asking follow-up questions. This is the standard expected in professional engineering teams and on Stack Overflow.

---

## Code Examples

**Reading Javadoc from the command line:**

```bash
# Generate Javadoc for your own code
javadoc -d docs/ src/main/java/com/example/*.java

# View the generated documentation
open docs/index.html   # macOS
xdg-open docs/index.html   # Linux
```

**A Javadoc comment in source code (what you will write and read daily):**

```java
/**
 * Calculates the compound interest for a given principal amount.
 *
 * <p>Uses the formula: A = P(1 + r/n)^(nt), where:
 * <ul>
 *   <li>P = principal amount</li>
 *   <li>r = annual interest rate (as a decimal, e.g., 0.05 for 5%)</li>
 *   <li>n = number of times interest is compounded per year</li>
 *   <li>t = time in years</li>
 * </ul>
 *
 * @param principal the initial amount of money (must be positive)
 * @param annualRate the annual interest rate as a decimal (must be >= 0)
 * @param compoundingFrequency the number of compounding periods per year (must be > 0)
 * @param years the time period in years (must be >= 0)
 * @return the total amount after compound interest is applied
 * @throws IllegalArgumentException if principal is negative or rate is negative
 * @since 1.0
 * @see <a href="https://en.wikipedia.org/wiki/Compound_interest">Compound Interest</a>
 */
public BigDecimal calculateCompoundInterest(
        BigDecimal principal,
        BigDecimal annualRate,
        int compoundingFrequency,
        double years) {
    if (principal.compareTo(BigDecimal.ZERO) < 0) {
        throw new IllegalArgumentException("Principal must be positive");
    }
    if (annualRate.compareTo(BigDecimal.ZERO) < 0) {
        throw new IllegalArgumentException("Annual rate must be non-negative");
    }
    // Implementation here
    return principal; // placeholder
}
```

Every public method in a professional Java codebase should have Javadoc like this. Your IDE (IntelliJ) renders it as formatted documentation when you hover over a method call.

**A minimal test (preview of what you will write in Phase 01):**

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class InterestCalculatorTest {

    @Test
    void shouldCalculateSimpleInterestCorrectly() {
        InterestCalculator calculator = new InterestCalculator();
        BigDecimal result = calculator.calculateSimpleInterest(
            new BigDecimal("1000.00"),
            new BigDecimal("0.05"),
            1
        );
        assertEquals(new BigDecimal("50.00"), result);
    }

    @Test
    void shouldThrowExceptionForNegativePrincipal() {
        InterestCalculator calculator = new InterestCalculator();
        assertThrows(IllegalArgumentException.class, () -> {
            calculator.calculateSimpleInterest(
                new BigDecimal("-100.00"),
                new BigDecimal("0.05"),
                1
            );
        });
    }
}
```

This is a unit test. It verifies that the code behaves correctly for both normal and edge cases. You will write hundreds of these.

---

## Important Notes

- Software engineering is a team sport. The code you write will be read by other engineers far more often than it will be executed by machines. Write for readability first, cleverness never.
- The SDLC model your team uses will shape your daily workflow. If you join a Scrum team, your work will be organized in 2-week sprints with ceremonies. If you join a Kanban team, work will flow continuously. Adapt to the team's process rather than insisting on a specific methodology.
- Version control is non-negotiable. There is no professional software development without Git. If you are not using Git, you are not engineering — you are scripting.
- Testing is not a phase that happens after development. It is part of development. Write tests as you write code, not as an afterthought. In fintech, untested code is a compliance risk.
- Deployment frequency is a strong indicator of engineering maturity. Teams that deploy once per quarter are fragile. Teams that deploy multiple times per day are resilient. Your goal is to build systems that can be deployed safely at any time.
- As a backend engineer, you will spend more time reading code than writing it. You will read your teammates' code during reviews, read library source code to understand behavior, and read legacy code to fix bugs. Invest in reading comprehension early.
- Java's dominance in fintech is not an accident. It is the result of decades of investment in correctness, performance, and stability. When you choose Java for a backend system, you are choosing a language that has been proven under the most demanding conditions in the industry.
- The ability to ask clear, well-structured technical questions will accelerate your learning faster than almost any other skill. Practice the format described above from day one.

---

## Practice

1. Write a one-paragraph definition of software engineering in your own words. Contrast it with programming. Add this to your glossary note.
2. Research the engineering blog of a fintech company (Stripe, Square, Revolut, or Adyen). Read one post about their backend architecture. Summarize it in 5-10 sentences in your Obsidian vault.
3. Open the Javadoc for `java.lang.String` at https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/String.html. Find the `substring` method. Read its documentation. Note the parameters, return type, exceptions, and the "Since" tag. Write down what you learned.
4. Write a "good question" following the format above for a hypothetical problem: you are trying to compile a Java file and get `error: cannot find symbol`. Include all the context, error output, and steps you tried. Add this to your notes as a template for future use.
5. List the five types of tests described in this section. For each, write one sentence describing when you would use it in a banking application.

---

## References

- Agile Manifesto: https://agilemanifesto.org/
- Scrum Guide: https://scrumguides.org/
- Spring Boot Documentation: https://docs.spring.io/spring-boot/docs/current/reference/html/
- Oracle Java Documentation: https://docs.oracle.com/en/java/
- JDK Enhancement Proposals: https://openjdk.org/jeps/0
- "The Pragmatic Programmer" by David Thomas and Andrew Hunt
- "Clean Code" by Robert C. Martin
