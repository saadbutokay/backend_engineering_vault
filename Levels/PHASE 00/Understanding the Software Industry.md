**Phase:** Level 0 - Environment & Mindset Setup  
**Date Studied:** 27th July, 2026

---
## What Problem Does This Solve?
Before writing a single line of code, you need to understand **what world you are entering**.

Most students start coding without knowing:
- What do companies actually need from engineers?
- What is backend vs frontend vs fullstack?
- What does a junior vs senior engineer actually do daily?
- Why Java? Why not something else?
- What does the industry in Bangladesh look like vs abroad?

If you skip this, you will study blindly.  
If you understand this first, **every topic you learn will have a purpose.**

---
## The Software Industry - Full Picture
The industry that can make or break you.

### What Is Software Engineering?
Software engineering is the process of **designing, building, testing, deploying, and maintaining software systems** that solve real problems for real people.

It is NOT just writing code.  
A professional engineer also:
- Thinks about **why** something should be built.
- Thinks about **how** it will break.
- Thinks about **who** else will read and maintain the code.
- Thinks about **scale** - will this work for 10 users? 10 million?
- Communicates with teammates, product managers, and sometimes clients

The Real Engineering Loop:
```
Understand Problem
      ↓
Design Solution
      ↓
Write Code
      ↓
Test It (did it work?)
      ↓
Deploy (put it live)
      ↓
Monitor (is it still working?)
      ↓
Fix / Improve
      ↓
(repeat forever)
```

---
### Types of Software Engineering Roles
You need to know these roles clearly because people use these terms in job postings every day.
#### Frontend Engineer
**What they build:**
- Everything the USER sees and interacts with.
- Buttons, pages, forms, animations, navigation.

**Technologies:**
- HTML, CSS, JavaScript, React, Vue, Angular

**They think about:**
- User experience, visual design, browser compatibility,
- page speed, accessibility

**Example work:**
- "Build the login page of bKash app"
- "Make the product page of Chaldal load faster"

---
#### Backend Engineer ← THIS IS YOU
**What they build:**
- Everything that happens BEHIND the scenes.
- The server, the database, the business logic,
- the APIs that the frontend calls.

**Technologies:**
- Java, Python, Go, Node.js, databases, cloud

**They think about:**
- Data integrity, security, performance, scalability,
- reliability, API design

**Example work:**
- "Build the API that processes bKash money transfers"
- "Build the system that tracks Chaldal inventory"
- "Build the authentication system that 1 million users log into"

The user never sees your work directly.
But without you, nothing works.

---
#### Fullstack Engineer
**What they build:**
- Both frontend AND backend.

**Common in:**
- Startups (small teams, everyone does everything)
- Smaller companies in Bangladesh

**Limitation:**
- Usually not as deep in either area as a specialist
- Big tech companies hire specialists

---
#### DevOps / Platform Engineer
**What they build:**
- The infrastructure that runs everything.
- Servers, pipelines, deployment systems, monitoring.

**Technologies:**
- Docker, Kubernetes, AWS, CI/CD, Terraform

**They think about:**
- "How do we deploy 50 times a day without breaking anything?"
- "How do we automatically scale when traffic spikes?"

---
#### Data Engineer
**What they build:**
- Pipelines that move, clean, and store massive amounts of data.

**Technologies:**
- Python, Spark, Kafka, data warehouses (BigQuery, Redshift)

**They think about:**
- Data reliability, data pipelines, analytics infrastructure
  
---
#### Machine Learning Engineer
**What they build:**
- Systems that use AI/ML models in production.

**Technologies:**
- Python, TensorFlow, PyTorch, MLflow

**They think about:**
- Model deployment, inference speed, model monitoring


> **Your Focus:** Backend Engineer → with awareness of DevOps and Data.  
> This is the highest-demand, highest-paid technical specialty  
> in Bangladesh and most of the world right now.

---
### Types of Companies You Can Work For
Understanding this helps you choose WHERE to apply and what to expect.

#### Product Companies
```
What they are:
  Companies that build their OWN software product.
  They hire engineers to build and improve that product.

Examples in Bangladesh:
  bKash, Nagad, Pathao, Shohoz, Chaldal, 10 Minute School

Examples abroad:
  Google, Amazon, Netflix, Shopify, Stripe, Spotify

Why they're better for engineers:
  ✓ You work on ONE product deeply — learn it well
  ✓ Better engineering culture and practices
  ✓ Higher salaries
  ✓ You see the full impact of your work
  ✓ Better learning environment

Downside:
  × You work in one tech stack mostly
  × Slower hiring (higher bar)
```

#### Software Houses / Outsourcing Companies
```
What they are:
  Companies that build software FOR other clients (not their own product).
  A client from Germany pays them to build a system.

Examples in Bangladesh:
  Brain Station 23, Kaz Software, TigerIT, DataSoft

Why they can be good:
  ✓ Easier to get first job
  ✓ Exposure to many different projects and domains
  ✓ Good for building portfolio early

Downside:
  × Lower salaries than product companies
  × Engineering culture varies a lot
  × Less depth on any one system
  × You may not know what you're building or why
```

#### Startups
```
What they are:
  New companies trying to grow fast.
  Small team, everyone does everything.

Why they can be good:
  ✓ Huge learning — you wear many hats
  ✓ More responsibility early
  ✓ Exciting if it succeeds

Downside:
  × Can be chaotic
  × Less mentorship
  × May not survive
  × Lower initial salary (sometimes offset by equity)
```

#### IT Departments of Large Corporations
```
What they are:
  Banks, telcos, government — they have internal IT teams.

Examples:
  Grameenphone IT, BRAC IT, Dutch-Bangla Bank IT

Why:
  ✓ Stable job, good salary for BD
  ✓ Established processes

Downside:
  × Slower technology adoption
  × Less exciting engineering challenges
  × Bureaucratic, slower pace
```

---
### What Is Backend Engineering - In Real Detail?
Let's make this concrete with a real example.

**Example: bKash Send Money Feature**
```
What the USER sees (Frontend):
  1. Opens bKash app
  2. Types recipient number
  3. Types amount
  4. Enters PIN
  5. Sees "Transfer Successful"

What YOU (Backend) build — every step:

  Step 1: User opens app
    → Mobile app connects to your server
    → Your server checks: is this user's session still valid?
    → Server returns user's account data

  Step 2: User types recipient number
    → App calls your API: GET /users/{phone}
    → Your server queries the database
    → Returns: "Yes, this number exists, name is Rahim"

  Step 3: User types amount and hits Send
    → App calls your API: POST /transactions
       Body: { from: "01711...", to: "01712...", amount: 500 }

  Step 4: Your server does the REAL work:
    → Validate: is amount > 0?
    → Validate: does sender have enough balance?
    → Validate: is this within daily limit?
    → Validate: is sender account not frozen?
    → Check for fraud patterns
    → Begin database transaction (atomic — all or nothing)
       → Deduct 500 + 5 charge from sender
       → Add 500 to recipient
       → Record transaction in history table
    → Commit transaction (make it permanent)
    → Publish event: "TransactionCompleted"
       → Notification service picks this up
       → Sends SMS to both sender and recipient
    → Return success response

  Step 5: "Transfer Successful" shown
    → Frontend receives your 200 OK response
    → Shows success screen

What happens if something fails?
    → If bank database is down: your server returns error
    → If amount is invalid: return 400 Bad Request
    → If transaction fails midway: rollback — NO money moves
    → All failures are logged so engineers can investigate
    → Monitoring alerts fire if error rate spikes

This entire invisible system = backend engineering.
```
This is what you are learning to build.

---
### Career Levels - Junior to Principal
This is the full career ladder. Know where you're going.
```
LEVEL 1: Junior / Associate Engineer (0-2 years)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
What you do:
  • Build features with guidance from senior engineers
  • Fix bugs
  • Write unit tests
  • Work on well-defined tasks
  • Learn the codebase and company systems

What you know:
  • Core language (Java)
  • Basic Spring Boot
  • SQL basics
  • Git
  • Can build a CRUD API

What you're evaluated on:
  • Can you complete tasks independently?
  • Is your code clean and readable?
  • Do you ask good questions?
  • Are you learning fast?

Salary in Bangladesh: 30,000 - 60,000 BDT/month
Salary abroad (entry): $50,000 - $80,000 USD/year

Your 1-year goal is to reach this level.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

LEVEL 2: Mid-Level Engineer (2-5 years)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
What you do:
  • Work independently on complete features
  • Design small systems
  • Mentor junior engineers
  • Participate in code reviews
  • Make technical decisions for your own tasks

What you know:
  • Deep Java + Spring Boot
  • Microservices basics
  • Kafka / messaging
  • Docker + basic Kubernetes
  • System design fundamentals
  • Performance awareness

What you're evaluated on:
  • Can you take a feature from idea to production independently?
  • Is your code production quality?
  • Can you debug complex issues?
  • Are you reliable?

Salary in Bangladesh: 80,000 - 150,000 BDT/month
Salary abroad: $90,000 - $140,000 USD/year

LEVEL 3: Senior Engineer (5-8 years)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
What you do:
  • Design entire systems
  • Own a domain end-to-end
  • Lead small teams technically
  • Define architecture for new features
  • Heavy code review
  • Oncall/production ownership

What you know:
  • Deep system design
  • Distributed systems understanding
  • Performance tuning
  • Full DevOps ownership
  • Security practices
  • Multiple languages/frameworks

What you're evaluated on:
  • Can you design systems that work at scale?
  • Can you make the entire team better?
  • Are you reliable under pressure?
  • Can you see problems before they happen?

Salary in Bangladesh: 150,000 - 300,000+ BDT/month
Salary abroad: $140,000 - $200,000 USD/year

LEVEL 4: Staff / Principal Engineer (8-15 years)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
What you do:
  • Set technical direction for multiple teams
  • Write engineering RFCs (proposals)
  • Identify systemic problems across the org
  • Define engineering standards
  • Work with VP/CTO level on strategy
  • Mentor senior engineers

What you know:
  • Everything about the systems your company runs
  • Industry-wide engineering trends
  • Business context deeply
  • Leadership and communication at a high level

What you're evaluated on:
  • Can you multiply the output of 10-20 engineers?
  • Can you make decisions that affect the company for years?
  • Is your judgment trusted across the organization?

Salary abroad: $200,000 - $400,000+ USD/year (FAANG level)

LEVEL 5: Distinguished / Fellow Engineer (15+ years)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Very rare. Industry-recognized technical leaders.
Think: Jeff Dean (Google), Linus Torvalds (Linux)
```

---
### Why Java?
You chose correctly. Here is the full justification.
```
REASON 1: Enterprise Dominance
  Java is the #1 language in enterprise software.
  Banks, fintech, e-commerce, healthcare — Java everywhere.
  bKash backend? Very likely Java.
  Most large Bangladesh companies? Java or .NET.
  Globally: Amazon, Netflix, LinkedIn, Uber — Java backends.

REASON 2: Spring Boot = Industry Standard
  Spring Boot is the most widely used Java backend framework.
  Knowing Spring Boot = knowing what 70%+ of Java job
  descriptions are asking for.

REASON 3: Mature Ecosystem
  Java has been around since 1995.
  Every problem has been solved. Every library exists.
  Kafka client: Java. Hibernate: Java. Spring: Java.
  Massive, battle-tested ecosystem.

REASON 4: Performance
  Modern Java (17, 21) is extremely fast.
  JIT compilation makes it competitive with C++
  for many workloads.
  Better than Python for high-throughput backends.

REASON 5: Jobs & Salary
  Java developers are among the highest paid globally.
  High demand, relatively lower supply vs JavaScript.
  Strong demand in Germany, Canada, Australia — all
  on your target list.

REASON 6: Strict Typing = Good Habits
  Java forces you to think about types, design, and structure.
  Engineers who start with Java build better habits
  than those who start with Python or JavaScript.
  When you learn Python next, it will feel easy.

REASON 7: OOP Foundation
  Java is fundamentally OOP.
  Learning OOP in Java properly prepares you for
  any other language or framework.

Java vs Alternatives:
  Java vs Python:
    Java wins on: performance, type safety, enterprise use
    Python wins on: speed of writing, data science, scripting
    → You're learning BOTH. Best of both worlds.

  Java vs JavaScript (Node.js):
    Java wins on: type safety, performance at scale, enterprise
    JavaScript wins on: frontend+backend same language, startup speed
    → Java is more valuable for serious backend roles.

  Java vs Go:
    Go wins on: simplicity, performance, cloud-native
    Java wins on: ecosystem, enterprise adoption, library support
    → Java first is the right call. Go can come later.

  Java vs C#:
    Very similar! Both are enterprise, strongly typed, mature.
    C# is Microsoft-stack. Java is everywhere else.
    → Java has wider industry adoption globally.
```

---
### Bangladesh Tech Market - Reality Check
```
Current State (2024-2025):
  • Growing fast — fintech (bKash, Nagad) leading
  • E-commerce growing (Chaldal, Shajgoj, Daraz BD)
  • Ride-sharing (Pathao, Shohoz) tech teams expanding
  • Remote work for international companies increasing
  • Most serious tech companies here use Java or Python

What Bangladesh companies pay (approximate 2024):
  Junior (0-1 yr):  30,000 - 60,000 BDT/month
  Mid (2-4 yr):     70,000 - 150,000 BDT/month
  Senior (5+ yr):   150,000 - 300,000+ BDT/month
  Remote USD job:   $2,000 - $6,000/month (life-changing in BD)

What Bangladesh companies want:
  Entry Level:
    ✓ Java OR Python (you'll have both)
    ✓ Spring Boot basics
    ✓ SQL
    ✓ Git
    ✓ REST APIs
    ✓ 1-2 real projects
    ✓ Decent English communication

  They DON'T necessarily want:
    × Perfect CGPA (2.83 is acceptable for most companies)
    × Years of experience
    × Formal certifications (but they help)

  Your 2.83 CGPA situation:
    Honest assessment:
    • Most BD product companies (bKash, Pathao etc.)
      will overlook CGPA if your projects are strong
    • Software houses (Brain Station 23 etc.) care less about CGPA
    • Pushing to 3.15 is still smart — important for MSc abroad
    • Your projects and GitHub will matter MORE than CGPA
      for getting your first job in Bangladesh

Tech Stack Most Common in BD Companies (2024):
  Backend:  Java (Spring Boot), Python (Django/FastAPI), PHP (legacy)
  Database: MySQL, PostgreSQL, Oracle
  Cloud:    AWS (growing fast), GCP
  Other:    Docker, Git, REST APIs
```

---
### What Does a Backend Engineer's Day Actually Look Like?
```
Morning (Standup Meeting):
  "Yesterday I finished the order creation API.
   Today I'm working on the payment processing integration.
   I'm blocked on getting test API credentials from the
   payments team."

Then you:
  • Open your ticket (Jira, Linear, Trello)
  • Pull latest code from GitHub
  • Write code in IntelliJ IDEA
  • Test locally (Postman, unit tests)
  • Push to GitHub
  • Create a Pull Request
  • A senior engineer reviews your code
  • You fix their comments
  • Code gets merged to main branch
  • CI/CD pipeline runs tests automatically
  • If tests pass, code deploys to staging server
  • QA team tests it
  • Deploy to production
  • Monitor for errors in logs/dashboard

Other things engineers do regularly:
  • Investigate production bugs ("the payment is failing
    for some users — why?")
  • Join technical discussions ("how should we design this new feature?")
  • Write documentation
  • Review other engineers' code
  • Attend architecture discussions

Key insight:
  Writing code is maybe 40-50% of the job.
  The rest is reading code, thinking, communicating,
  reviewing, debugging, and documenting.
```

---
### The Tools You Will Use Every Day
_You don't need to learn all of these right now.  
Just know they exist and what each one does._

```
WRITING CODE:
  IntelliJ IDEA    → Where you write Java code (IDE)
  VS Code          → Where you'll write Python, YAML, etc.

VERSION CONTROL:
  Git              → Track all code changes
  GitHub           → Store code online, collaborate

BUILDING & RUNNING:
  Maven / Gradle   → Build your Java project, manage libraries
  JDK 21           → The Java runtime that actually runs your code

TESTING APIS:
  Postman          → Send HTTP requests to test your APIs
  curl             → Same thing but in terminal

DATABASES:
  PostgreSQL       → The database (stores your data)
  DBeaver          → Visual tool to see and query your database
  Redis            → Fast in-memory cache database

CONTAINERS:
  Docker           → Package your app so it runs anywhere
  Docker Compose   → Run multiple containers together (app + db + redis)

CLOUD:
  AWS              → Where your app will live in production

MONITORING:
  Grafana          → See dashboards of how your app is performing
  Prometheus       → Collect metrics from your app

COMMUNICATION (at companies):
  Slack / Teams    → Team communication
  Jira / Linear    → Task tracking (what are you building?)
  Confluence / Notion → Documentation
```

---
## Interview Questions

> These will not appear in a first-round interview but knowing them  
> shows you have perspective — interviewers love that.

```
Q: Why did you choose Java as your primary language?
A: Java is the dominant language in enterprise backend development, especially in fintech and e-commerce — which are growing fast both in Bangladesh and globally. Spring Boot is the industry standard framework, and Java's strong typing forces good engineering habits. It also has the widest job market for backend roles internationally.

Q: What is the difference between frontend and backend engineering?
A: Frontend engineers build what users see and interact with, the visual interface. Backend engineers build the server-side logic, APIs, databases, and business rules that power the application. When a user sends money on bKash, the button is frontend, the validation, transaction processing, and data storage is backend.

Q: What type of company would you like to work for and why?
A: I'm targeting product companies where I can work on a real system at scale, go deep on one domain, and see the direct impact of my engineering work on real users.
```

---
## Common Misconceptions (Things Most Students Believe That Are Wrong)
```
❌ "I need to know everything before I apply for jobs"
✅ Companies hire juniors to LEARN. Apply when you have
   solid fundamentals + 1-2 real projects.

❌ "My CGPA of 2.83 means I can't get a good job"
✅ CGPA matters less in tech than in most fields.
   A strong GitHub + good projects + communication skills
   beats a 4.0 CGPA with no practical skills every time.
   (For MSc abroad, CGPA matters more — work on it.)

❌ "I need to learn every language and framework"
✅ Go DEEP on one stack first. Java + Spring Boot.
   Companies want depth, not breadth, at junior level.

❌ "Senior engineers write more code than juniors"
✅ Senior engineers write LESS code but think MORE.
   Their job is design, reviews, and making the team
   effective — not grinding out features.

❌ "LeetCode is all you need to get a job"
✅ LeetCode helps pass interviews but you need real
   project experience to back it up. Both matter.

❌ "I'll learn backend engineering by watching tutorials"
✅ You learn by BUILDING. Watch tutorial → close it →
   build something similar from scratch. That's how it works.

❌ "Bangladesh tech jobs aren't worth it — I'll go abroad immediately"
✅ 1-2 years of solid experience in Bangladesh makes you
   a much stronger MSc applicant AND gives you real
   engineering experience to talk about in interviews abroad.
```

---
## Key Takeaways
```
1. Backend engineering = the invisible engine that makes apps work.
   Users never see it. Nothing works without it.

2. You are entering a real profession, not just learning to code.
   Think like an engineer from day one.

3. Java is the right choice.
   Spring Boot is the right framework.
   PostgreSQL is the right database to learn first.
   These three alone can get you a job.

4. Career levels are clear:
   Junior → Mid → Senior → Staff → Principal
   You are aiming for Junior in 1 year.
   That is an achievable and respectable goal.

5. Bangladesh has real opportunities.
   bKash, Pathao, Chaldal are legitimate tech companies.
   A 1-2 year experience there → MSc abroad is a real path.

6. Your CGPA is not your destiny in this field.
   Your GitHub is.

7. A backend engineer's job is not just writing code.
   It's designing, testing, debugging, deploying, monitoring,
   communicating, and continuously improving systems.

8. Start thinking in systems, not in lines of code.
```

---