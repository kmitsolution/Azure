# Module 1 — Lecture 1

## Azure Databases Explained: Complete Database Landscape in Azure

This is the **foundation lecture** for the entire Azure Database series. The objective is not to create every database today, but to make students understand **what database options Azure provides, why there are so many, and how to choose the right one**.

---

# 1. Learning Objectives

By the end of this lecture, students should understand:

* What a database is
* Why applications need databases
* SQL vs NoSQL
* Relational vs non-relational databases
* Managed vs unmanaged databases
* Azure database service landscape
* Azure SQL Database vs SQL Managed Instance vs SQL Server VM
* MySQL and PostgreSQL
* Cosmos DB
* Redis
* Azure Storage Tables
* Azure Data Explorer
* When to choose each service
* Basic Azure Portal navigation
* Basic Azure CLI discovery commands
* Real-world database architecture

---

# 2. What Is a Database?

Start with a very simple example.

Suppose we build an e-commerce application:

```text
                    E-Commerce Application
                            |
             +--------------+--------------+
             |              |              |
          Customers       Products       Orders
             |              |              |
             +--------------+--------------+
                            |
                         DATABASE
```

The application needs to store:

* Customer information
* Product information
* Orders
* Payments
* Inventory
* Addresses
* Reviews

We don't want this information to disappear when the application stops.

That's where a **database** comes in.

### Simple definition

> A database is an organized system for storing, managing, retrieving, and modifying data.

---

# 3. Why Do We Need a Database?

Imagine storing customers in a text file:

```text
customer.txt
```

```text
Raman, Bangalore, 987654321
John, London, 123456789
David, New York, 456789123
```

This works for a few records.

But imagine:

```text
10 Million Customers
100 Million Orders
1 Billion Product Transactions
```

We need:

* Fast searching
* Multiple users accessing data
* Security
* Transactions
* Backup
* Recovery
* High availability
* Scaling
* Data consistency

That's why we use database management systems.

---

# 4. What Is a DBMS?

**DBMS = Database Management System**

Examples:

* SQL Server
* MySQL
* PostgreSQL
* Oracle
* MongoDB
* Cassandra

The DBMS provides functionality for:

```text
Application
     |
     ↓
    DBMS
     |
     ↓
   Data
```

For example:

```text
.NET Application
       |
       ↓
SQL Server
       |
       ↓
Employee Database
```

---

# 5. Two Major Database Categories

The first major decision is:

```text
                    DATABASES
                       |
             +---------+---------+
             |                   |
          RELATIONAL          NoSQL
             |                   |
            SQL             Non-Relational
```

---

# 6. Relational Databases

Relational databases store data in **tables**.

Example:

### Customer table

| CustomerId | Name  | City      |
| ---------: | ----- | --------- |
|        101 | Raman | Bangalore |
|        102 | John  | London    |
|        103 | David | New York  |

### Orders table

| OrderId | CustomerId | Amount |
| ------: | ---------: | -----: |
|    5001 |        101 |   2500 |
|    5002 |        102 |   4500 |

Notice:

```text
Customer
   |
CustomerId
   |
Orders
```

This relationship is where the term **relational database** comes from.

---

# 7. What Is SQL?

SQL means:

**Structured Query Language**

Example:

```sql
SELECT *
FROM Customers
WHERE City = 'Bangalore';
```

Insert:

```sql
INSERT INTO Customers
VALUES (104, 'Alex', 'Mumbai');
```

Update:

```sql
UPDATE Customers
SET City = 'Delhi'
WHERE CustomerId = 104;
```

Delete:

```sql
DELETE FROM Customers
WHERE CustomerId = 104;
```

---

# 8. Examples of Relational Databases

Common relational databases include:

```text
SQL Server
MySQL
PostgreSQL
Oracle
MariaDB
```

In Azure, we'll primarily study:

```text
                    RELATIONAL
                        |
          +-------------+-------------+
          |             |             |
       Azure SQL      MySQL       PostgreSQL
          |
    +-----+------+
    |            |
SQL Database   Managed Instance
```

And also:

**SQL Server running on Azure VM**

---

# 9. What Is NoSQL?

NoSQL generally means **non-relational database technologies**.

Instead of forcing data into traditional tables, NoSQL databases can use models such as:

* Document
* Key-value
* Graph
* Wide-column

For example, a document database might store:

```json
{
  "id": "101",
  "name": "Raman",
  "city": "Bangalore",
  "skills": [
    "Azure",
    "AWS",
    "DevOps"
  ]
}
```

This is different from a traditional SQL table.

---

# 10. Why Do We Need NoSQL?

Imagine a social media application.

A user might have:

```text
Name
Age
Location
Friends
Followers
Posts
Likes
Comments
Photos
Videos
Interests
```

The structure can change frequently.

NoSQL databases can be useful when applications need:

* Flexible schemas
* Very large scale
* High throughput
* Global distribution
* Low-latency access

---

# 11. Azure Database Landscape

Now introduce the students to the Azure ecosystem.

Think of Azure databases like this:

```text
                         AZURE DATABASES
                                |
       +------------------------+-------------------------+
       |                        |                         |
   RELATIONAL                 NoSQL                   SPECIALIZED
       |                        |                         |
       |                        |                         |
  +----+----+              Cosmos DB                 Redis
  |    |    |                                          |
 SQL  MySQL PostgreSQL                              Cache
  |
  +-------------------+
  |                   |
SQL Database      SQL Managed Instance
  |
SQL Server VM
```

There are also specialized data services such as:

* Azure Data Explorer
* Azure Table Storage
* Azure Storage
* Azure Synapse/Fabric for analytics workloads

---

# 12. Azure SQL Database

Azure SQL Database is a **fully managed relational database service** based on the SQL Server engine.

Architecture:

```text
              Azure SQL Database
                     |
             +-------+-------+
             |               |
         SQL Engine      Database
                             |
                    Tables / Views
                    Stored Procedures
                    Indexes
```

Microsoft manages much of the underlying infrastructure.

You generally don't manage:

* Physical server hardware
* Operating system patching
* SQL Server installation
* Basic infrastructure maintenance

---

# 13. Azure SQL Managed Instance

Managed Instance provides a broader SQL Server compatibility surface than Azure SQL Database while remaining a managed Azure service.

Think:

```text
SQL Server
    |
    | More compatibility
    ↓
Managed Instance
    |
    | More cloud-native management
    ↓
Azure SQL Database
```

A common migration scenario:

```text
On-Premises SQL Server
          |
          ↓
Azure SQL Managed Instance
```

We'll study this deeply later.

---

# 14. SQL Server on Azure VM

Here Azure provides the VM infrastructure, while you have much more control over the operating system and SQL Server installation/configuration.

```text
             Azure VM
                |
        +-------+-------+
        |               |
       OS          SQL Server
```

You are responsible for significantly more:

* Windows/Linux OS
* SQL Server configuration
* Patching
* Maintenance
* Storage configuration
* SQL Server administration

But you get much more control.

---

# 15. The Three SQL Choices

This is one of the most important concepts of this course.

| Feature                   | Azure SQL Database          | SQL Managed Instance  | SQL Server on VM        |
| ------------------------- | --------------------------- | --------------------- | ----------------------- |
| Managed service           | Yes                         | Yes                   | No, VM-based            |
| OS access                 | No                          | No                    | Yes                     |
| SQL Server compatibility  | High, but database-oriented | Very high             | Full                    |
| Infrastructure management | Low                         | Low                   | High                    |
| Control                   | Lower                       | Medium                | Highest                 |
| Best for                  | Cloud applications          | SQL Server migrations | Full SQL Server control |

A simple decision model:

```text
Do I need full OS control?
       |
      YES
       ↓
 SQL Server VM

       NO
       |
       ↓
Do I need broad SQL Server instance compatibility?
       |
      YES
       ↓
 Managed Instance

       NO
       ↓
Azure SQL Database
```

---

# 16. Azure Database for MySQL

MySQL is one of the world's most popular open-source relational databases.

Azure provides:

**Azure Database for MySQL — Flexible Server**

Architecture:

```text
Application
    |
    ↓
Azure MySQL Flexible Server
    |
    ↓
MySQL Database
    |
    +---- Tables
    +---- Indexes
    +---- Users
```

Typical use cases:

* PHP applications
* WordPress
* Web applications
* Open-source applications
* Custom applications

---

# 17. Azure Database for PostgreSQL

PostgreSQL is another powerful open-source relational database.

Azure provides:

**Azure Database for PostgreSQL — Flexible Server**

Example:

```text
Python Application
       |
       ↓
PostgreSQL Flexible Server
       |
       ↓
Database
```

PostgreSQL is popular for:

* Enterprise applications
* GIS
* Analytics-related workloads
* Python applications
* Advanced SQL workloads
* Applications requiring PostgreSQL-specific features

---

# 18. Cosmos DB

Now introduce the major NoSQL service.

**Azure Cosmos DB** is Microsoft's globally distributed database platform.

Basic architecture:

```text
                    Cosmos DB Account
                           |
                     Database
                           |
                      Container
                           |
                  +--------+--------+
                  |        |        |
                Item     Item     Item
```

Cosmos DB can store JSON-like documents using its NoSQL API.

Example:

```json
{
  "id": "1001",
  "product": "Laptop",
  "price": 75000,
  "category": "Electronics"
}
```

---

# 19. Cosmos DB APIs

This is important because Cosmos DB isn't limited to one data model/API.

You'll encounter:

```text
Cosmos DB
    |
    +--- NoSQL API
    |
    +--- MongoDB API
    |
    +--- Cassandra
    |
    +--- Gremlin
    |
    +--- Table
```

We'll dedicate several lectures to Cosmos DB later.

---

# 20. Azure Cache for Redis

Redis is primarily an **in-memory data store/cache**, rather than a replacement for your primary relational database in every scenario.

Example:

```text
                Application
                    |
          +---------+---------+
          |                   |
       Redis              Database
          |                   |
     Fast Access          Permanent Data
```

Suppose an application frequently asks:

```text
What are the top 10 products?
```

Instead of querying the database every time:

```text
Application
    ↓
Redis
    ↓
Response
```

This can reduce database load and improve response time.

---

# 21. Azure Table Storage

Azure Table Storage provides a simple NoSQL key-value/entity model.

Conceptually:

```text
Storage Account
      |
     Table
      |
 +----+----+
 |         |
Partition Row
 Key      Key
```

Useful for certain:

* Simple structured data
* Large-scale key/value workloads
* Cost-sensitive storage scenarios

---

# 22. Azure Data Explorer

Azure Data Explorer is designed for analyzing large volumes of data, particularly:

* Logs
* Telemetry
* Time-series data
* Application monitoring
* IoT data

Example:

```text
IoT Devices
    |
    ↓
Telemetry
    |
    ↓
Azure Data Explorer
    |
    ↓
KQL Queries
    |
    ↓
Reports / Analysis
```

We'll later introduce **KQL — Kusto Query Language**.

---

# 23. Transactional vs Analytical Databases

Another important concept.

### OLTP

**Online Transaction Processing**

Example:

```text
Customer places order
       ↓
Inventory updated
       ↓
Payment recorded
       ↓
Order created
```

Typical databases:

* Azure SQL
* MySQL
* PostgreSQL

### OLAP

**Online Analytical Processing**

Example:

```text
Sales data
    ↓
Millions of transactions
    ↓
Analytics
    ↓
Business reports
```

Different Azure services can be more appropriate depending on the analytical workload.

---

# 24. Managed vs Unmanaged

This is a fundamental Azure concept.

### Unmanaged / IaaS

```text
Azure VM
   |
   +-- OS
   +-- SQL Server
   +-- Database
```

You manage:

```text
Hardware abstraction
OS
Database software
Configuration
Patching
Backup strategy
```

### Managed / PaaS

```text
Azure SQL
```

Azure manages much of the underlying platform.

You primarily focus on:

```text
Database
Data
Security
Queries
Configuration
Performance
```

---

# 25. Azure Database Decision Tree

Give students this diagram.

```text
                    START
                      |
                      ↓
             What type of data?
                      |
            +---------+---------+
            |                   |
        Relational            NoSQL
            |                   |
      +-----+------+        Cosmos DB
      |     |      |
     SQL   MySQL PostgreSQL
      |
 +----+----------------+
 |                     |
Need full SQL       Cloud-native
Server control?       database?
 |                     |
YES                   YES
 |                     |
SQL VM              Azure SQL DB
 |
NO
 |
Need instance-level
SQL compatibility?
 |
YES
 |
Managed Instance
```

---

# 26. Azure Portal — Database Services

For this introductory lecture, don't create resources yet.

Instead show students how to explore the services.

Go to:

**Azure Portal → Create a resource**

Search for:

```text
SQL Database
```

Then:

```text
Azure Database for MySQL
```

Then:

```text
Azure Database for PostgreSQL
```

Then:

```text
Azure Cosmos DB
```

Then:

```text
Azure Cache for Redis
```

The purpose of today's portal exercise is simply:

> **Identify the database services available in Azure.**

---

# 27. Azure CLI — Discover Database Services

First make sure you're logged in:

```bash
az login
```

Check your subscription:

```bash
az account show
```

List subscriptions:

```bash
az account list -o table
```

Set a subscription:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

---

# 28. Explore Azure SQL CLI

Check available SQL commands:

```bash
az sql --help
```

List SQL servers:

```bash
az sql server list -o table
```

List databases:

```bash
az sql db list \
  --resource-group <RESOURCE_GROUP> \
  --server <SQL_SERVER> \
  -o table
```

---

# 29. Explore MySQL CLI

```bash
az mysql --help
```

For Flexible Server:

```bash
az mysql flexible-server --help
```

List MySQL Flexible Servers:

```bash
az mysql flexible-server list -o table
```

---

# 30. Explore PostgreSQL CLI

```bash
az postgres --help
```

Flexible Server:

```bash
az postgres flexible-server --help
```

List servers:

```bash
az postgres flexible-server list -o table
```

---

# 31. Explore Cosmos DB CLI

```bash
az cosmosdb --help
```

List Cosmos DB accounts:

```bash
az cosmosdb list -o table
```

---

# 32. Important CLI Concept

Students should understand that Azure CLI follows a hierarchy.

For example:

```text
az
 |
 +-- sql
 |    |
 |    +-- server
 |    |
 |    +-- db
 |
 +-- mysql
 |    |
 |    +-- flexible-server
 |
 +-- postgres
 |    |
 |    +-- flexible-server
 |
 +-- cosmosdb
```

So when learning Azure CLI, don't memorize hundreds of commands.

Understand the hierarchy.

---

# 33. Real-World Case Study

Let's imagine **KMIT E-Commerce**.

The application has:

```text
Customers
Products
Orders
Payments
Product Catalog
Session Data
Analytics
```

A possible architecture:

```text
                         USERS
                           |
                           ↓
                    Web Application
                           |
             +-------------+-------------+
             |                           |
             ↓                           ↓
        Azure SQL                    Redis Cache
             |
       Transaction Data
             |
       +-----+------+
       |            |
 Customers       Orders
 Products        Payments
```

For a product catalog with flexible document data:

```text
Application
     |
     ↓
Cosmos DB
     |
Product Catalog
```

For analytics:

```text
Application / IoT / Logs
          |
          ↓
   Azure Data Explorer
```

So one application can legitimately use **multiple data services**.

---

# 34. Important Concept — There Is No "Best Database"

Students often ask:

> "Which database is best in Azure?"

The correct answer is:

> **It depends on the workload.**

For example:

| Requirement                     | Possible Choice     |
| ------------------------------- | ------------------- |
| SQL Server application          | Azure SQL           |
| SQL Server migration            | Managed Instance    |
| Full OS/SQL control             | SQL VM              |
| Open-source relational          | MySQL               |
| Advanced open-source relational | PostgreSQL          |
| Global NoSQL                    | Cosmos DB           |
| Extremely fast cache            | Redis               |
| Simple key/value entities       | Table Storage       |
| Telemetry/log analytics         | Azure Data Explorer |

---

# 35. Beginner Practical Lab

## Lab Objective

Explore Azure's database services.

### Task 1

Open Azure Portal.

Navigate to:

```text
Create a resource
```

Search for:

```text
SQL Database
MySQL
PostgreSQL
Cosmos DB
Azure Cache for Redis
```

Record:

* Service name
* Database type
* Pricing model
* Region availability
* Networking options

---

### Task 2 — CLI

Run:

```bash
az sql --help
```

```bash
az mysql flexible-server --help
```

```bash
az postgres flexible-server --help
```

```bash
az cosmosdb --help
```

---

### Task 3 — Architecture

Draw this architecture:

```text
                 Azure Databases
                       |
       +---------------+---------------+
       |               |               |
    Relational       NoSQL         Specialized
       |               |               |
   +---+---+       Cosmos DB       Redis
   |   |   |
 Azure MySQL PostgreSQL
 SQL
```

---

# 36. Assignment 1

### Azure Database Service Selection

A company has the following requirements.

### Requirement 1

A .NET application uses SQL Server and requires a relational database.

**Question:** Which Azure service would you select?

---

### Requirement 2

A company wants to migrate an existing SQL Server environment with minimal application changes and needs broad SQL Server compatibility.

**Question:** Which service would you investigate first?

---

### Requirement 3

A company requires full Windows Server and SQL Server administrative control.

**Question:** Which option?

---

### Requirement 4

A startup has a Python application and wants PostgreSQL.

**Question:** Which Azure service?

---

### Requirement 5

A global gaming application requires a highly scalable NoSQL database with low-latency access across regions.

**Question:** Which Azure service?

---

### Requirement 6

An application repeatedly reads the same session information and needs very fast access.

**Question:** Which service could be used as a cache?

---

### Requirement 7

IoT devices generate huge amounts of telemetry that needs time-series/log-style analysis.

**Question:** Which Azure service would you consider?

---

# 37. Interview Questions

### Beginner

**Q1. What is a database?**

A structured system used to store, manage, retrieve and modify data.

**Q2. What is DBMS?**

Database Management System.

**Q3. What is the difference between SQL and NoSQL?**

SQL databases generally use structured relational tables and SQL; NoSQL databases use non-relational models such as documents, key-value, graph or wide-column structures.

**Q4. What is Azure SQL Database?**

A fully managed relational database service based on the SQL Server engine.

**Q5. What is Azure SQL Managed Instance?**

A managed Azure SQL service providing broad SQL Server instance-level compatibility for many workloads.

---

### Intermediate

**Q6. Azure SQL Database vs SQL Managed Instance?**

Azure SQL Database is more database-centric and cloud-native; Managed Instance provides broader SQL Server compatibility and instance-level capabilities.

**Q7. Azure SQL Database vs SQL Server on Azure VM?**

Azure SQL Database is managed PaaS; SQL Server on Azure VM gives much more control because you manage the VM and SQL Server.

**Q8. Why would you choose PostgreSQL instead of Azure SQL?**

If the application requires PostgreSQL-specific functionality, compatibility, ecosystem or organizational standards.

**Q9. When would you use Cosmos DB?**

For suitable globally distributed, highly scalable NoSQL workloads where flexible data models and low-latency access are important.

**Q10. Is Redis a database replacement?**

Not generally. Redis is commonly used as a high-speed in-memory data store/cache, although it supports broader data-store use cases.

---

# 38. Scenario-Based Interview Questions

### Scenario 1

> You have an old SQL Server application. The application depends on SQL Server features that aren't available in Azure SQL Database. What would you investigate?

**Answer:** Azure SQL Managed Instance, or SQL Server on Azure VM if full OS/SQL control is required.

---

### Scenario 2

> Your application needs global distribution and a flexible JSON data model.

**Answer:** Evaluate Cosmos DB.

---

### Scenario 3

> Your application requires extremely fast access to frequently requested data.

**Answer:** Consider Redis as a caching layer.

---

### Scenario 4

> The company says, "We want SQL Server in Azure, but we don't want to manage the operating system."

**Answer:** Consider Azure SQL Database or SQL Managed Instance depending on compatibility requirements.

---

# 39. Key Takeaways

Students should leave Lecture 1 remembering this:

```text
                 AZURE DATABASES
                       |
          +------------+------------+
          |                         |
      RELATIONAL                  NoSQL
          |                         |
   +------+------+              Cosmos DB
   |      |      |
 Azure  MySQL PostgreSQL
  SQL
   |
 +--+----------------+
 |                   |
SQL Database     Managed Instance
 |
SQL Server VM
```

And:

```text
Need full control?
       ↓
    SQL VM

Need SQL Server compatibility?
       ↓
Managed Instance

Cloud-native relational database?
       ↓
Azure SQL Database

MySQL?
       ↓
MySQL Flexible Server

PostgreSQL?
       ↓
PostgreSQL Flexible Server

Global NoSQL?
       ↓
Cosmos DB

Fast cache?
       ↓
Redis
```

---

## Recommended next lecture

### **Module 1 — Lecture 2: Azure Database Architecture & Core Concepts**

In Lecture 2, we should go one level deeper into:

* Database
* Database server
* Instance
* Tables
* Schema
* Database engine
* Compute
* Storage
* Connections
* Authentication
* Authorization
* Regions
* Availability Zones
* Networking
* Public vs Private access
* PaaS vs IaaS
* **How an application actually connects to an Azure database**

Then we'll create our **first real Azure SQL Database from the Portal and Azure CLI**, rather than jumping into SQL commands too early.
