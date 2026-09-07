# Module 1 — Lecture 4

# Azure SQL Compute, DTU, vCore, Serverless, Service Tiers & Pricing

This lecture is one of the **most important Azure SQL fundamentals** because when creating an Azure SQL Database, students immediately encounter choices like:

* DTU
* vCore
* General Purpose
* Business Critical
* Hyperscale
* Serverless
* Provisioned compute
* Storage

The goal is to understand **what these options mean, when to use them, and how to select them** rather than simply clicking a SKU in the Portal.

---

# 1. Learning Objectives

By the end of this lecture, students should understand:

* What database compute means
* CPU, memory and I/O
* DTU purchasing model
* vCore purchasing model
* DTU vs vCore
* Provisioned compute
* Serverless compute
* General Purpose
* Business Critical
* Hyperscale
* Storage
* Scaling up/down
* Scale-up vs scale-out
* Cost considerations
* Portal configuration
* Azure CLI commands
* Real-world SKU selection
* Interview questions

---

# 2. Start With the Basic Question

When we created:

```text
EmployeeDB
```

Azure asked us:

> **How much compute and storage do you want?**

Why?

Because a database needs resources to process requests.

Consider:

```text
100 Users
   ↓
Database
```

versus:

```text
100,000 Users
   ↓
Database
```

The second workload may require significantly more:

* CPU
* Memory
* I/O
* Connections
* Storage

Therefore Azure provides different compute configurations.

---

# 3. Database Compute

Think of compute as the **engine power** available to the database.

```text
                 DATABASE
                    |
             +------+------+
             |             |
            CPU          Memory
             |
             +------+
                    |
                   I/O
```

When users execute:

```sql
SELECT *
FROM Employees;
```

the database engine needs compute resources to process that request.

---

# 4. CPU

CPU performs calculations and query processing.

For example:

```sql
SELECT
    Department,
    AVG(Salary)
FROM Employees
GROUP BY Department;
```

The database needs CPU to:

* Read data
* Perform calculations
* Group records
* Produce the result

High CPU can indicate:

```text
Heavy queries
Too many requests
Inefficient queries
Insufficient compute
```

---

# 5. Memory

Memory is used for things such as:

* Query execution
* Caching data
* Sorting
* Joins
* Database operations

Conceptually:

```text
Disk / Storage
      |
      ↓
   Memory
      |
      ↓
 Database Engine
      |
      ↓
    Query
```

More memory can allow more useful data to remain available for fast processing, depending on workload and service configuration.

---

# 6. I/O

I/O means:

**Input / Output**

Database workloads frequently involve reading and writing data.

For example:

```text
Application
     |
     ↓
INSERT
     |
     ↓
Database
     |
     ↓
Storage
```

Or:

```text
SELECT
  ↓
Read Data
  ↓
Return Result
```

Database performance isn't only about CPU.

A workload can be limited by:

```text
CPU
Memory
I/O
Query design
Indexes
Concurrency
```

---

# 7. Azure SQL Purchasing Models

Historically, Azure SQL Database has used two major ways to describe compute:

```text
Azure SQL Database
       |
       +----------------+
       |                |
      DTU             vCore
```

### DTU

**Database Transaction Unit**

### vCore

**Virtual Core**

For modern Azure architecture discussions, **vCore is generally the more transparent model to understand and use**, especially when comparing compute capacity across workloads.

---

# 8. What Is DTU?

DTU combines several resource dimensions into one performance unit.

Conceptually:

```text
                 DTU
                  |
        +---------+---------+
        |         |         |
       CPU      Memory      I/O
```

Microsoft defines DTU as a blended measure of CPU, memory, reads and writes.

So instead of individually selecting resources, you choose a DTU performance level.

For example:

```text
Basic
S0
S1
S2
...
```

The exact available configurations depend on the Azure SQL offering and current platform options.

---

# 9. Why DTU Was Created

DTU simplifies database sizing for beginners.

Instead of asking:

```text
How many CPU cores?
How much memory?
What I/O capability?
```

you can think:

```text
I need a higher DTU performance level.
```

So:

```text
More DTU
   ↓
More overall database performance capacity
```

But DTU hides the individual resource dimensions.

---

# 10. DTU Limitation

Suppose your workload is:

```text
CPU-heavy
```

DTU doesn't let you reason about CPU independently as clearly as vCore.

That's one reason vCore is often preferred for more detailed capacity planning.

---

# 11. What Is vCore?

vCore means:

**Virtual CPU core**

Instead of using one combined performance number, the vCore model exposes compute more directly.

Conceptually:

```text
Database
   |
   +---- vCores
   |
   +---- Memory
   |
   +---- Storage
```

For example:

```text
2 vCores
4 vCores
8 vCores
16 vCores
```

The exact memory and performance characteristics depend on the selected service tier/hardware generation.

---

# 12. DTU vs vCore

This is an important interview question.

| DTU                                  | vCore                                |
| ------------------------------------ | ------------------------------------ |
| Blended performance unit             | Compute expressed in virtual cores   |
| Simpler                              | More transparent                     |
| Less granular resource comparison    | Better for capacity planning         |
| Existing/legacy workloads may use it | Common choice for modern deployments |
| Easier for beginners                 | Better for architecture decisions    |

A simple way to explain it:

> **DTU tells you the overall performance capacity as a combined unit; vCore gives you a more infrastructure-oriented view of compute.**

---

# 13. Provisioned Compute

Now we come to another important concept.

With **provisioned compute**, resources are allocated for the database whether or not the database is continuously busy.

Conceptually:

```text
                    Provisioned
                        |
            +-----------+-----------+
            |                       |
        Database Busy          Database Idle
            |                       |
        Uses Compute           Compute remains
                               provisioned
```

This is useful for workloads that run continuously.

Example:

```text
Production Banking Application
       ↓
Database
       ↓
24 × 7 workload
```

---

# 14. Serverless Compute

Serverless is designed for workloads with variable or intermittent usage.

Conceptually:

```text
              Serverless
                  |
        +---------+---------+
        |                   |
    Workload HIGH       Workload LOW
        |                   |
   More compute         Less compute
```

Azure SQL Database serverless can dynamically adjust compute within configured limits based on workload.

For suitable workloads, it can also **auto-pause** after a configured period of inactivity, reducing compute consumption during idle periods.

---

# 15. Serverless Example

Imagine a development database:

```text
Developer
   |
   ↓
Uses DB
9 AM - 6 PM
```

At night:

```text
6 PM
 ↓
No activity
 ↓
Database becomes idle
```

Serverless can be attractive for this type of intermittent workload.

But:

> **Serverless is not automatically cheaper for every workload.**

For a continuously busy production database, provisioned compute may be more appropriate.

---

# 16. Provisioned vs Serverless

| Provisioned                    | Serverless                                  |
| ------------------------------ | ------------------------------------------- |
| Predictable compute allocation | Dynamically adjusts compute                 |
| Good for steady workloads      | Good for variable/intermittent workloads    |
| Typically always provisioned   | Can auto-pause when configured and eligible |
| Predictable performance        | Startup/resume behavior must be considered  |
| Common production choice       | Useful for dev/test and variable workloads  |

---

# 17. Service Tiers

Now we move to service tiers.

For Azure SQL Database, major service-tier concepts include:

```text
Azure SQL Database
       |
       +-- General Purpose
       |
       +-- Business Critical
       |
       +-- Hyperscale
```

These are **not simply different CPU sizes**.

They represent different architectural/performance characteristics.

---

# 18. General Purpose

General Purpose is designed for many common workloads.

Think:

```text
                    General Purpose
                          |
            +-------------+-------------+
            |             |             |
         Compute       Storage       Networking
```

Typical use cases:

* Business applications
* Web applications
* APIs
* Standard OLTP
* Development/production workloads where extreme I/O performance isn't required

---

# 19. Business Critical

Business Critical is designed for workloads requiring higher performance and stronger availability characteristics.

Typical scenarios:

```text
High transaction volume
Low latency
Mission-critical workloads
High I/O requirements
```

Examples:

```text
Financial Application
Payment System
High-volume ERP
```

The architecture differs from General Purpose and can provide higher performance characteristics.

---

# 20. General Purpose vs Business Critical

Think of it like this:

```text
General Purpose
      |
      ↓
Balanced
Cost + Performance


Business Critical
      |
      ↓
Higher Performance
Lower Latency
Mission-Critical Workloads
```

Don't select Business Critical merely because it sounds better.

Ask:

> **Does my workload actually require those capabilities?**

---

# 21. Hyperscale

Hyperscale is designed for very large databases and workloads requiring significant scalability.

Think:

```text
                    Hyperscale
                        |
                +-------+-------+
                |               |
            Compute          Storage
                |               |
                |        Distributed architecture
                |
          Read Scale Options
```

It is particularly useful for large database workloads and scenarios where storage and scaling requirements are beyond typical database sizes.

---

# 22. When Would We Consider Hyperscale?

Imagine:

```text
Database
   |
   ↓
10 GB
```

General Purpose may be perfectly reasonable.

But consider:

```text
Database
   |
   ↓
Several TB
   |
   ↓
Rapid growth
   |
   ↓
High transaction volume
```

Now Hyperscale becomes a service tier worth evaluating.

---

# 23. Service Tier Comparison

| Feature                  | General Purpose       | Business Critical                      | Hyperscale                      |
| ------------------------ | --------------------- | -------------------------------------- | ------------------------------- |
| General workloads        | Excellent             | Yes                                    | Yes                             |
| High-performance OLTP    | Good                  | Excellent                              | Excellent                       |
| Large databases          | Yes                   | Yes                                    | Strong fit                      |
| High I/O workloads       | Good                  | Very strong                            | Very strong                     |
| Specialized architecture | No                    | Yes                                    | Yes                             |
| Cost                     | Lower                 | Higher                                 | Workload-dependent              |
| Best fit                 | Most common workloads | Mission-critical low-latency workloads | Large/highly scalable databases |

Exact limits, supported features, and pricing vary by Azure region, hardware generation, configuration, and service evolution.

---

# 24. Storage

When creating a database, you'll also configure storage.

Think:

```text
Database
   |
   +---- Compute
   |
   +---- Storage
```

Storage includes data such as:

```text
Tables
Indexes
Database structures
Transaction-related data
```

Depending on the service tier, Azure manages storage architecture differently.

---

# 25. Storage ≠ Performance

Very important.

Suppose:

```text
Database = 1 TB
```

That doesn't mean:

```text
1 TB = High Performance
```

You could have:

```text
Large Storage
+
Insufficient Compute
=
Poor Performance
```

Or:

```text
Adequate Compute
+
Poorly Designed Queries
=
Poor Performance
```

Performance is a system problem, not just a storage-size problem.

---

# 26. Scale Up vs Scale Down

Suppose we have:

```text
2 vCores
```

and our workload increases.

We can scale up:

```text
2 vCores
    ↓
4 vCores
    ↓
8 vCores
```

This is **vertical scaling** or **scale up**.

If workload decreases:

```text
8 vCores
    ↓
4 vCores
    ↓
2 vCores
```

That's scaling down.

---

# 27. Scale Out

Scale out means adding capacity by distributing workload across multiple resources/instances where the architecture supports it.

Conceptually:

```text
              Application
                   |
        +----------+----------+
        |          |          |
       DB1        DB2        DB3
```

Azure SQL has specific scale-out patterns/features, such as readable replicas for suitable workloads, but scale-out is not simply "add three primary databases."

This distinction becomes important when we discuss:

* Read scale
* Replication
* Geo-replication
* Hyperscale
* Application architecture

---

# 28. Practical Scenario

Imagine an online shopping application.

Normal traffic:

```text
9 AM
 ↓
1,000 users
```

During a sale:

```text
8 PM
 ↓
100,000 users
```

The database workload increases dramatically.

Possible actions include:

```text
Scale compute
Optimize queries
Add indexes
Use caching
Use read scale where appropriate
Review application architecture
```

Don't simply increase the SKU without investigating the actual bottleneck.

---

# 29. Portal — View Compute Configuration

Open:

```text
Azure Portal
   ↓
SQL Databases
   ↓
EmployeeDB
```

Look for:

```text
Compute + storage
```

You'll see the current configuration.

Depending on the current Azure Portal experience, you may see options involving:

```text
Service tier
Compute tier
Hardware configuration
Compute size
vCores
Data max size
```

The exact UI can change over time.

---

# 30. Portal — Scale the Database

Go to:

```text
EmployeeDB
    ↓
Compute + storage
```

You can review available configurations.

For example, conceptually:

```text
Current:

General Purpose
4 vCores

Change to:

General Purpose
8 vCores
```

Then:

```text
Apply / Save
```

Azure performs the requested configuration change.

**Important:** Scaling can have operational impact. Don't randomly change production databases during peak traffic.

---

# 31. Portal — Serverless

If the selected configuration supports serverless, you'll see compute configuration options related to:

```text
Compute tier:
Serverless
```

You may configure:

```text
Minimum vCores
Maximum vCores
Auto-pause delay
```

Example:

```text
Minimum = 0.5 vCore
Maximum = 4 vCores
Auto-pause = configured inactivity period
```

These are **illustrative values**; choose based on the workload and currently supported options.

---

# 32. When NOT to Use Serverless

Avoid assuming serverless is ideal for:

```text
24 × 7 production workload
```

especially when:

* The database is continuously active
* Predictable latency is important
* Workload is consistently high
* Auto-pause/resume behavior isn't appropriate

Instead, provisioned compute may be more suitable.

---

# 33. CLI — Check Database Configuration

Run:

```bash
az sql db show \
  --resource-group <RESOURCE_GROUP> \
  --server <SQL_SERVER> \
  --name EmployeeDB \
  -o json
```

To see selected properties:

```bash
az sql db show \
  --resource-group <RESOURCE_GROUP> \
  --server <SQL_SERVER> \
  --name EmployeeDB \
  --query "{name:name,status:status,sku:sku,capacity:sku.capacity,tier:sku.tier}" \
  -o table
```

This gives you a useful view of the database SKU configuration.

---

# 34. CLI — Change the SKU

You can change the database SKU using:

```bash
az sql db update \
  --resource-group <RESOURCE_GROUP> \
  --server <SQL_SERVER> \
  --name EmployeeDB \
  --service-objective S1
```

For vCore-based configurations, use the appropriate current Azure CLI parameters/options for the selected service tier and compute model.

This is one reason you should always check:

```bash
az sql db update --help
```

before using a command copied from an old tutorial.

---

# 35. CLI — Check Available Options

Use:

```bash
az sql db list-editions \
  --location centralindia \
  -o table
```

This helps explore available editions/service objectives.

You can also inspect CLI documentation:

```bash
az sql db create --help
```

and:

```bash
az sql db update --help
```

---

# 36. CLI — Example Serverless Concept

The exact CLI syntax depends on the current Azure SQL API/CLI version and selected tier.

Before deploying, check:

```bash
az sql db create --help
```

and:

```bash
az sql db update --help
```

Look for parameters related to:

```text
compute model
min capacity
max capacity
auto pause
```

**Teaching point:** Azure CLI evolves. Don't teach students to memorize old command syntax; teach them how to discover the currently supported command options.

---

# 37. How Should We Select a SKU?

Never start with:

> "Which SKU is cheapest?"

Start with:

### Question 1

What is the workload?

```text
Dev/Test
Small business
Production
Mission critical
Large database
```

### Question 2

Is workload:

```text
Constant
or
Variable?
```

### Question 3

What performance is required?

```text
Low
Medium
High
Extreme
```

### Question 4

How much data?

```text
10 GB
500 GB
2 TB
10 TB+
```

### Question 5

What availability requirements exist?

```text
Normal
High
Mission Critical
```

---

# 38. SKU Decision Tree

Use this in your lecture.

```text
                  DATABASE WORKLOAD
                         |
                         ↓
                 Is it intermittent?
                    /          \
                  YES           NO
                   |             |
                   ↓             ↓
              Consider        Provisioned
              Serverless       Compute
                   |             |
                   +------↓------+
                          |
                    Performance?
                          |
              +-----------+-----------+
              |                       |
          Standard              Mission Critical
              |                       |
              ↓                       ↓
      General Purpose         Business Critical
              |
              |
        Very large DB /
        specialized scale
              |
              ↓
          Hyperscale
```

This is a conceptual decision tree, not a substitute for checking the current Azure SKU/feature matrix.

---

# 39. Real-World Case Study #1

### College Website

Traffic:

```text
500 users/day
```

Database:

```text
20 GB
```

Workload:

```text
Mostly daytime
```

Recommendation:

> Start with a low-cost configuration appropriate for development/small production workloads, then measure actual usage.

Don't deploy Business Critical simply because it is "faster."

---

# 40. Case Study #2 — E-Commerce

Traffic:

```text
50,000 users/day
```

Database:

```text
500 GB
```

Workload:

```text
24 × 7
```

Need:

```text
Predictable performance
High availability
```

Possible starting point:

> Evaluate provisioned **General Purpose**, then benchmark and monitor the workload.

If latency/I/O requirements are significantly higher, evaluate Business Critical.

---

# 41. Case Study #3 — Financial Application

Requirements:

```text
Very low latency
High transaction volume
Business critical
```

Possible choice:

> Evaluate **Business Critical**.

But the final choice should still be based on actual performance, availability, and compliance requirements.

---

# 42. Case Study #4 — Development Database

Requirement:

```text
Developer uses database
only during working hours
```

Workload:

```text
Intermittent
```

Possible choice:

> Evaluate **Serverless** if the database/service configuration and workload are suitable.

---

# 43. Case Study #5 — Huge Database

Requirement:

```text
Several TB
Rapid growth
Large-scale workload
```

Possible choice:

> Evaluate **Hyperscale**.

---

# 44. Cost Optimization

Database cost is influenced by several factors.

Conceptually:

```text
             DATABASE COST
                  |
       +----------+----------+
       |          |          |
    Compute    Storage    Other usage
       |
       ↓
    vCores /
    service tier
```

Depending on service/configuration, other cost components can include:

* Storage
* Backup retention
* Networking
* Additional replicas/features
* Monitoring/logging
* Other Azure dependencies

---

# 45. Golden Rule of Database Cost

Don't optimize only for:

> **Lowest price**

Optimize for:

> **Lowest cost that satisfies the business requirements.**

Example:

```text
$20 database
+
Application unavailable
+
Poor performance
=
Expensive
```

A slightly more expensive database that meets the SLA can be cheaper for the business overall.

---

# 46. Performance Testing

Before production, test.

Architecture:

```text
Application
    |
    ↓
Load Test
    |
    ↓
Azure SQL
    |
    ↓
Monitor
```

Measure:

```text
CPU
IO
Latency
DTU / vCore utilization
Connections
Query duration
```

Then adjust the SKU.

---

# 47. Don't Solve Every Performance Problem by Scaling

Suppose CPU is 95%.

You could:

```text
2 vCores
   ↓
8 vCores
```

But maybe the real problem is:

```sql
SELECT *
FROM Orders
```

returning millions of rows.

Better solution may involve:

```text
Index
Query optimization
Pagination
Filtering
Caching
Architecture changes
```

This becomes our later **Azure SQL Performance module**.

---

# 48. Interview Questions

### Q1. What is DTU?

A blended Azure SQL Database performance measure combining CPU, memory, reads and writes.

### Q2. What is vCore?

A virtual CPU core used to express compute capacity in the vCore purchasing model.

### Q3. DTU vs vCore?

DTU provides a combined performance measure, while vCore exposes compute capacity more directly and supports more infrastructure-oriented capacity planning.

### Q4. What is serverless?

A compute model that dynamically adjusts compute capacity based on workload within configured limits and can auto-pause after inactivity when supported/configured.

### Q5. What is provisioned compute?

Compute capacity that is allocated for the database rather than dynamically scaling down to zero during idle periods.

---

# 49. More Interview Questions

### Q6. What is General Purpose?

A balanced service tier suitable for many common database workloads.

### Q7. What is Business Critical?

A service tier designed for workloads requiring higher performance, lower latency, and stronger availability/performance characteristics.

### Q8. What is Hyperscale?

A service tier designed for very large databases and highly scalable workloads using a specialized architecture.

### Q9. Does more storage automatically mean better performance?

No.

### Q10. Does more vCore automatically solve slow queries?

No.

---

# 50. Scenario Interview Question

> **Your Azure SQL database has 90% CPU utilization. What would you do?**

Don't answer:

> "Increase the vCores."

A better answer:

```text
1. Check CPU metrics
2. Identify expensive queries
3. Review Query Performance Insight / monitoring
4. Check execution plans
5. Check indexes
6. Check concurrency
7. Optimize queries
8. Then consider scaling compute
```

---

# 51. Another Scenario

> **A development database is used only from 9 AM to 6 PM. Would you automatically choose provisioned compute?**

Not necessarily.

Evaluate:

```text
Serverless
```

because the workload is intermittent.

But check:

* Auto-pause support
* Minimum/maximum compute
* Resume behavior
* Application connection behavior
* Actual cost

---

# 52. Another Scenario

> **A production database is busy 24×7 and requires predictable performance. Serverless or provisioned?**

Generally:

> **Evaluate provisioned compute first.**

The final decision depends on the actual workload and requirements.

---

# 53. Hands-On Lab

## Lab: Change Azure SQL Compute

Take the `EmployeeDB` created in Lecture 3.

### Step 1

Open:

```text
Azure Portal
 ↓
SQL Databases
 ↓
EmployeeDB
 ↓
Compute + storage
```

Record:

```text
Current Tier:
Current Compute:
Current Storage:
```

### Step 2

Explore:

```text
General Purpose
Business Critical
Hyperscale
```

Don't necessarily deploy each one—just inspect the options and understand the differences.

### Step 3

If your lab budget permits, change to another inexpensive supported configuration.

### Step 4

Run:

```bash
az sql db show \
  --resource-group <RG> \
  --server <SERVER> \
  --name EmployeeDB \
  --query "{name:name,sku:sku,tier:sku.tier,capacity:sku.capacity}" \
  -o json
```

### Step 5

Compare Portal vs CLI output.

---

# 54. Assignment — SKU Selection

Choose a suitable Azure SQL approach for each.

### Scenario A

```text
Student project
10 GB database
Used 3 hours/day
Low budget
```

### Scenario B

```text
E-commerce
500 GB
24×7
Moderate traffic
```

### Scenario C

```text
Banking
High transaction volume
Very low latency
Mission critical
```

### Scenario D

```text
Enterprise database
Several TB
Rapid growth
Large workload
```

### Scenario E

```text
Developer environment
Idle most of the day
Occasional workload spikes
```

For each scenario answer:

```text
Service Tier:
Compute Model:
Reason:
```

---

# 55. Assignment — Practical

Using your Azure SQL Database:

1. Find the current compute configuration.
2. Find the current service tier.
3. Find the current storage configuration.
4. Find whether serverless is available for your configuration.
5. Run the database for a test workload.
6. Monitor CPU.
7. Monitor database utilization.
8. Scale the database if appropriate.
9. Verify the new configuration with Azure CLI.
10. Return the database to the desired lab configuration.

---

# 56. One Important AZ-104/AZ-305 Concept

Students should not memorize:

```text
S0 = this
S1 = this
```

as the main learning objective.

Instead, teach them:

```text
Workload
   ↓
Requirements
   ↓
Compute model
   ↓
Service tier
   ↓
Compute capacity
   ↓
Storage
   ↓
Cost
   ↓
Performance testing
```

That's much closer to how real architecture decisions are made.

---

# 57. Final Whiteboard

End the lecture with this:

```text
                     AZURE SQL DATABASE
                             |
                +------------+------------+
                |                         |
            Compute                    Storage
                |
        +-------+-------+
        |               |
    Provisioned      Serverless
        |
        ↓
     vCore / DTU
        |
        ↓
   Service Tier
        |
   +----+----+----+
   |         |    |
General   Business Hyper-
Purpose   Critical scale
```

And the decision:

```text
                WORKLOAD
                   |
                   ↓
        Continuous or variable?
             /            \
           /                \
     Continuous           Variable
         |                   |
   Provisioned           Serverless*
         |
         ↓
  Performance needs
         |
   +-----+------+
   |            |
Standard     Critical
   |            |
   ↓            ↓
General      Business
Purpose      Critical
                |
                ↓
          Very large /
          specialized
                |
                ↓
            Hyperscale
```

`*` Serverless suitability depends on the workload and current service capabilities.

---

# 🎯 Lecture 4 Takeaway

The students should now be able to answer:

> **"How do I decide what Azure SQL configuration to choose?"**

The answer is:

**Don't start with the SKU. Start with the workload.**

```text
Workload
   ↓
Performance
   ↓
Availability
   ↓
Database Size
   ↓
Traffic Pattern
   ↓
Compute Model
   ↓
Service Tier
   ↓
Cost
```

### Next Lecture — Module 1, Lecture 5

**Azure SQL Database Networking — Public Access, Firewall, Private Endpoint, VNet, Private DNS & Service Endpoint**

We'll take the `EmployeeDB` from today's lab and build the networking piece **hands-on**. We'll start with the public endpoint and firewall, then progressively convert the architecture to:

```text
                    Azure VNet
                       |
                 Application VM
                       |
                       ↓
                Private Endpoint
                       |
                 Private IP
                       |
                  Azure SQL
                       |
                  Private DNS
```

This will also connect directly with the **NIC/NSG/VNet concepts** you've already been teaching in your Azure course.
