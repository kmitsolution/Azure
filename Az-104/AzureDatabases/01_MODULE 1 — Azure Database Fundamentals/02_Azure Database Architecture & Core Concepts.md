# Module 1 — Lecture 2

# Azure Database Architecture & Core Concepts

This lecture is important because before creating Azure SQL, MySQL, PostgreSQL, or Cosmos DB resources, students need to understand **what actually exists behind an Azure database service**.

The key question we want students to answer is:

> **When my application connects to an Azure database, what exactly is happening between the application and the data?**

---

# 1. Learning Objectives

By the end of this lecture, students will understand:

* Database
* Database server
* Database instance
* Database engine
* Schema
* Tables
* Compute
* Storage
* Connection
* Authentication
* Authorization
* Public vs Private access
* Region
* Availability Zone
* PaaS vs IaaS
* Azure Resource Group
* Subscription
* How an application connects to a database
* Basic Azure database architecture

---

# 2. Start With a Simple Architecture

Let's begin with something students already understand.

Suppose we have:

```text
User
  |
  ↓
Web Application
  |
  ↓
Database
```

For example:

```text
Browser
   |
   ↓
ASP.NET Application
   |
   ↓
Azure SQL Database
   |
   ↓
Customers
Orders
Products
```

The application doesn't normally communicate directly with a physical disk.

There are several layers in between.

---

# 3. Complete Database Architecture

A simplified architecture is:

```text
                    USER
                      |
                      ↓
                APPLICATION
                      |
                      ↓
                 NETWORK
                      |
                      ↓
              DATABASE SERVICE
                      |
             +--------+--------+
             |                 |
          COMPUTE            STORAGE
             |                 |
             ↓                 ↓
       Database Engine      Data Files
             |
             ↓
          DATABASE
             |
       +-----+-----+
       |           |
     Schema      Objects
                   |
        +----------+----------+
        |          |          |
      Tables     Views      Indexes
```

This architecture is extremely useful for understanding Azure databases.

---

# 4. What Is a Database?

A **database** is a logical collection of data.

For example:

```text
EmployeeDB
```

could contain:

```text
EmployeeDB
 |
 +-- Employees
 |
 +-- Departments
 |
 +-- Salaries
 |
 +-- Locations
```

Another database could be:

```text
SalesDB
 |
 +-- Customers
 +-- Orders
 +-- Products
```

So:

> **Database = logical container for related data and database objects.**

---

# 5. What Is a Database Server?

A database server is the service/infrastructure that hosts and processes database workloads.

In traditional SQL Server:

```text
SQL Server Machine
       |
       +-- SQL Server Instance
               |
               +-- Database
               +-- Database
               +-- Database
```

In Azure, the terminology varies by service.

For example, Azure SQL Database commonly has a logical SQL server resource that provides things such as:

* Server-level administration
* Firewall configuration
* Authentication settings
* Hosting context for databases

Important:

> An Azure SQL logical server is **not the same thing as a traditional physical SQL Server machine**.

---

# 6. What Is a Database Instance?

An **instance** is an execution environment for a database engine.

Traditional SQL Server can have:

```text
Server
 |
 +-- SQL Server Instance 1
 |       |
 |       +-- DB1
 |       +-- DB2
 |
 +-- SQL Server Instance 2
         |
         +-- DB3
         +-- DB4
```

This distinction becomes particularly important when comparing:

```text
Azure SQL Database
       vs
Azure SQL Managed Instance
       vs
SQL Server on Azure VM
```

---

# 7. Database Engine

The database engine is the software responsible for processing database operations.

For SQL Server:

```text
Application
    |
    ↓
SQL Query
    |
    ↓
SQL Server Database Engine
    |
    ↓
Data
```

For MySQL:

```text
Application
    ↓
MySQL Engine
    ↓
Database
```

For PostgreSQL:

```text
Application
    ↓
PostgreSQL Engine
    ↓
Database
```

For Cosmos DB, the underlying architecture and APIs are different because it is a distributed NoSQL platform.

---

# 8. What Is a Schema?

A schema is a logical namespace/container for database objects.

For example, SQL Server might have:

```text
EmployeeDB
 |
 +-- dbo
 |    |
 |    +-- Employees
 |    +-- Departments
 |
 +-- HR
      |
      +-- Salaries
```

Here:

```text
dbo
HR
```

are schemas.

You can reference an object as:

```sql
SELECT *
FROM dbo.Employees;
```

---

# 9. What Is a Table?

A table stores structured relational data.

Example:

```text
Employees
+----+--------+------------+
| ID | Name   | Department |
+----+--------+------------+
| 1  | Raman  | IT         |
| 2  | John   | HR         |
| 3  | David  | Finance    |
+----+--------+------------+
```

A table contains:

* Rows
* Columns

---

# 10. Database Objects

A relational database doesn't contain only tables.

It can contain:

```text
Database
 |
 +-- Tables
 +-- Views
 +-- Indexes
 +-- Stored Procedures
 +-- Functions
 +-- Triggers
 +-- Constraints
 +-- Users
 +-- Roles
```

Students should understand these because we'll work with them extensively in later lectures.

---

# 11. Compute vs Storage

This is one of the most important cloud concepts.

A database requires:

### Compute

Used to process:

* SQL queries
* Transactions
* Connections
* Calculations

### Storage

Used to store:

* Tables
* Indexes
* Database files
* Logs
* Data

Conceptually:

```text
             DATABASE
                 |
        +--------+--------+
        |                 |
      Compute           Storage
        |                 |
    Processing          Data
    Queries             Files
    Connections         Indexes
```

---

# 12. Why Compute Matters

Suppose your application suddenly gets:

```text
100 users
```

Then:

```text
10,000 users
```

Then:

```text
1,000,000 users
```

The database may need more compute capacity.

Therefore Azure database services provide different compute options.

For example, Azure SQL uses concepts such as:

* vCore
* DTU
* Provisioned compute
* Serverless

We'll study these in detail later.

---

# 13. Why Storage Matters

Suppose:

```text
Database = 10 GB
```

After one year:

```text
Database = 500 GB
```

Eventually:

```text
Database = 2 TB
```

The database needs sufficient storage.

But remember:

> **Storage capacity and compute performance are related concepts, but they are not the same thing.**

A database can have plenty of storage and still have poor performance because compute resources are insufficient.

---

# 14. Database Connection

How does an application communicate with a database?

Example:

```text
Application
     |
     | TCP/IP
     |
     ↓
Database Endpoint
     |
     ↓
Database Service
```

A connection normally involves:

```text
Hostname
Port
Username / Identity
Password / Token
Database Name
Encryption Settings
```

For SQL Server, the traditional port is:

```text
1433
```

MySQL:

```text
3306
```

PostgreSQL:

```text
5432
```

These are common defaults, though configuration can vary.

---

# 15. Example Connection String

For an application connecting to SQL Server/Azure SQL, you might see something conceptually like:

```text
Server=tcp:<server>.database.windows.net,1433;
Database=EmployeeDB;
User ID=<username>;
Password=<password>;
Encrypt=True;
```

The exact connection string depends on the driver and authentication method.

---

# 16. DNS Is Involved

Suppose the application connects to:

```text
myserver.database.windows.net
```

The application needs to resolve that hostname.

Conceptually:

```text
Application
    |
    ↓
DNS
    |
    ↓
IP Address
    |
    ↓
Azure Database
```

This becomes extremely important when we introduce:

**Private Endpoint + Private DNS Zone.**

---

# 17. Public Database Connectivity

A simplified public architecture:

```text
                  Internet
                     |
                     ↓
              Public Endpoint
                     |
                  Firewall
                     |
                     ↓
               Azure Database
```

Example:

```text
Laptop
   |
Internet
   |
Public endpoint
   |
Azure SQL
```

The database can be configured to allow or deny network access.

---

# 18. Private Database Connectivity

Now consider a production application.

```text
              Azure VNet
        +-----------------------+
        |                       |
        |      Application      |
        |          |            |
        |          ↓            |
        |   Private Endpoint    |
        |          |            |
        +----------|------------+
                   |
                   ↓
             Azure Database
```

The database can be accessed through a private IP address.

Conceptually:

```text
Application
     |
     ↓
10.x.x.x
     |
     ↓
Private Endpoint
     |
     ↓
Azure Database
```

We'll build this practically later.

---

# 19. Public vs Private

| Feature           | Public Access                    | Private Access                                    |
| ----------------- | -------------------------------- | ------------------------------------------------- |
| Internet endpoint | Yes                              | No direct public endpoint required                |
| Private IP        | Not necessarily                  | Yes                                               |
| Firewall          | Important                        | Still relevant depending on service/configuration |
| VNet              | Not necessarily                  | Yes                                               |
| Private Endpoint  | No                               | Yes                                               |
| Common use        | Development / selected scenarios | Production / restricted environments              |

---

# 20. Authentication

Authentication answers:

> **Who are you?**

For example:

```text
Application
    |
    ↓
"Who are you?"
    |
    ↓
Username/password
```

Or:

```text
Application
    |
    ↓
Microsoft Entra ID
    |
    ↓
Identity token
```

Common authentication approaches include:

* SQL authentication
* Microsoft Entra authentication
* Managed identity

---

# 21. Authorization

Authorization answers:

> **What are you allowed to do?**

For example:

```text
User
 |
 +-- SELECT
 |
 +-- INSERT
 |
 +-- UPDATE
 |
 X-- DROP DATABASE
```

Authentication:

```text
Who are you?
```

Authorization:

```text
What can you do?
```

This distinction is extremely important for Azure security.

---

# 22. Azure RBAC vs Database Permissions

Students frequently confuse these.

### Azure RBAC

Controls access to **Azure resources**.

Example:

```text
Subscription
   |
Resource Group
   |
Azure SQL Server
```

Roles:

* Reader
* Contributor
* Owner

### Database permissions

Control what someone can do **inside the database**.

Example:

```text
SELECT
INSERT
UPDATE
DELETE
EXECUTE
```

Therefore:

> **Azure RBAC and SQL database permissions are different authorization layers.**

---

# 23. Resource Hierarchy

Azure resources exist inside a hierarchy.

```text
Microsoft Entra Tenant
        |
        ↓
    Subscription
        |
        ↓
   Resource Group
        |
        ↓
      Resource
```

Example:

```text
Subscription
 |
 +-- rg-production
       |
       +-- Azure SQL Server
       |
       +-- SQL Database
       |
       +-- Key Vault
       |
       +-- VNet
```

---

# 24. Region

When creating an Azure database, you generally select an Azure region.

Examples:

```text
Central India
South India
East US
West Europe
```

Region selection affects:

* Latency
* Data residency
* Availability
* Disaster recovery architecture
* Cost

For example:

```text
Users in India
      |
      ↓
Central India Database
```

usually gives lower network latency than placing the database far away, all else being equal.

---

# 25. Availability Zones

Some Azure regions provide Availability Zones.

Conceptually:

```text
                Azure Region
                     |
       +-------------+-------------+
       |             |             |
      AZ1           AZ2           AZ3
       |             |             |
    Compute       Compute       Compute
    /Data         /Data         /Data
```

Availability Zones are physically separate locations within an Azure region designed to improve resiliency against datacenter-level failures.

Whether a particular database service, tier, and configuration supports zone redundancy depends on the service and SKU.

---

# 26. Region vs Availability Zone

Don't confuse them.

### Region

Geographical Azure location:

```text
Central India
```

### Availability Zone

Separate datacenter/zone within a supported region:

```text
Central India
 |
 +-- Zone 1
 +-- Zone 2
 +-- Zone 3
```

---

# 27. High Availability

High Availability means:

> The service remains available despite certain failures.

Conceptually:

```text
              Application
                   |
                   ↓
             Database Service
                /       \
               /         \
          Instance A   Instance B
```

The exact implementation differs by Azure database service.

---

# 28. Disaster Recovery

HA and DR are not the same.

### High Availability

Usually protects against failures within a region/service architecture.

### Disaster Recovery

Protects against larger failures such as a regional outage.

Example:

```text
Central India
     |
 Primary Database
     |
     | Replication
     ↓
Another Azure Region
     |
 Secondary Database
```

We'll later cover:

* Geo-replication
* Failover groups
* Backup
* Point-in-time restore
* RPO
* RTO

---

# 29. RPO and RTO

Very important interview concepts.

### RPO

**Recovery Point Objective**

> How much data can the business afford to lose?

Example:

```text
RPO = 15 minutes
```

Means the organization targets recovery with no more than approximately 15 minutes of data loss, depending on the solution.

### RTO

**Recovery Time Objective**

> How quickly must the application be restored?

Example:

```text
RTO = 1 hour
```

---

# 30. PaaS vs IaaS

Now connect this to Azure databases.

## IaaS

SQL Server on Azure VM:

```text
Azure
 |
 VM
 |
 OS
 |
 SQL Server
 |
 Database
```

You manage much of the stack.

---

## PaaS

Azure SQL Database:

```text
Azure
 |
 Managed Database Service
 |
 Database
```

Azure manages much of the infrastructure.

---

# 31. Responsibility Comparison

```text
                 IaaS             PaaS
              SQL VM         Azure SQL Database
--------------------------------------------------
Hardware        Azure              Azure
Networking      Shared             Shared
OS              YOU                Azure
SQL Server      YOU                Azure
DB              YOU                YOU
Data            YOU                YOU
Users           YOU                YOU
Queries         YOU                YOU
```

This is a simplified responsibility model; exact responsibilities vary by service.

---

# 32. Database Architecture Example

Let's build a realistic architecture.

```text
                         USERS
                           |
                           ↓
                    Web Application
                           |
                           ↓
                      Azure VNet
                           |
             +-------------+-------------+
             |                           |
        Application VM              Private DNS
             |                           |
             +-------------+-------------+
                           |
                           ↓
                   Private Endpoint
                           |
                           ↓
                    Azure SQL Database
                           |
                  +--------+--------+
                  |                 |
               Database          Backup
                  |
          +-------+-------+
          |       |       |
       Tables   Indexes  Views
```

This architecture will become the basis for later hands-on labs.

---

# 33. Azure Portal — Explore the Architecture

For this lecture, let's create **no paid database resources yet**.

Open:

**Azure Portal → All resources**

Then inspect any existing Azure SQL resource if you already have one.

Look for:

```text
Overview
Networking
Security
Compute + Storage
Backup
Monitoring
```

The exact menu names can vary by Azure service and portal updates.

---

# 34. Portal Exercise — Identify the Layers

If you have an Azure SQL Database:

### Step 1

Open:

```text
Azure Portal
    ↓
SQL databases
```

### Step 2

Select your database.

### Step 3

Identify:

```text
Resource Group
Subscription
Region
Server
Database
Compute
Storage
Networking
```

### Step 4

Open:

```text
Networking
```

Identify:

* Public network access
* Firewall rules
* Private endpoint options

---

# 35. Azure CLI Exercise

Check your account:

```bash
az account show -o table
```

List resource groups:

```bash
az group list -o table
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

Get database details:

```bash
az sql db show \
  --resource-group <RESOURCE_GROUP> \
  --server <SQL_SERVER> \
  --name <DATABASE_NAME>
```

---

# 36. CLI — Inspect SQL Server

Run:

```bash
az sql server show \
  --resource-group <RESOURCE_GROUP> \
  --name <SQL_SERVER>
```

Look at properties such as:

```text
location
fullyQualifiedDomainName
administratorLogin
publicNetworkAccess
```

This is a good exercise for students to understand that the Azure Portal is simply a graphical interface over Azure resource management APIs.

---

# 37. Important Concept — Portal vs CLI

Students should understand:

```text
Azure Portal
     |
     ↓
Azure Resource Manager
     |
     ↓
Azure Resource
```

and:

```text
Azure CLI
     |
     ↓
Azure Resource Manager
     |
     ↓
Azure Resource
```

Therefore:

> Portal and CLI are two different ways of managing Azure resources.

Later we'll add:

```text
PowerShell
Bicep
Terraform
REST API
```

---

# 38. Hands-On Lab

## Lab: Understand Azure Database Architecture

### Task 1

Find an Azure SQL Database in your subscription.

Record:

```text
Database Name:
Resource Group:
Region:
Server:
FQDN:
Compute Model:
Service Tier:
Public Network Access:
```

### Task 2

Run:

```bash
az sql db show \
  --resource-group <RG> \
  --server <SERVER> \
  --name <DB> \
  -o table
```

### Task 3

Run:

```bash
az sql server show \
  --resource-group <RG> \
  --name <SERVER> \
  -o json
```

Identify:

* Location
* FQDN
* Public network access
* Administrator configuration

---

# 39. Assignment — Draw the Architecture

Create an architecture diagram for:

> A web application running on an Azure VM needs to connect securely to an Azure SQL Database without exposing the database publicly.

Your diagram must contain:

```text
User
 ↓
Azure VM
 ↓
VNet
 ↓
Private Endpoint
 ↓
Azure SQL Database
```

Also add:

```text
Private DNS Zone
```

Then answer:

1. Why do we need a private endpoint?
2. Why do we need DNS?
3. What is the difference between authentication and authorization?
4. What is compute?
5. What is storage?
6. What is a region?
7. What is an Availability Zone?
8. What is RPO?
9. What is RTO?
10. Why is Azure SQL Database considered PaaS?

---

# 40. Interview Questions

### Q1. What is the difference between a database and a database server?

A database is a logical collection of data and objects. A database server/service provides the environment that processes and hosts database workloads.

### Q2. What is a database engine?

The software component responsible for processing database operations such as queries and transactions.

### Q3. What is compute in a database?

Compute provides processing resources for queries, transactions, connections and other database operations.

### Q4. What is storage?

Storage provides persistent capacity for database data, indexes, logs and related files.

### Q5. Authentication vs authorization?

```text
Authentication = Who are you?
Authorization  = What can you do?
```

### Q6. What is Azure RBAC?

Azure Role-Based Access Control manages permissions to Azure resources.

### Q7. Does Azure RBAC automatically grant SELECT permission inside SQL?

**No.** Azure RBAC and database-level permissions are separate concepts.

### Q8. What is a private endpoint?

A private endpoint provides a private network interface/IP in a VNet for accessing supported Azure services privately.

### Q9. Region vs Availability Zone?

A region is an Azure geographical deployment location. An Availability Zone is a physically separate zone within a supported Azure region.

### Q10. PaaS vs IaaS?

PaaS provides a managed platform where Azure manages much of the underlying infrastructure. IaaS gives you more control, such as OS-level control on an Azure VM, but also more management responsibility.

---

# 41. Scenario-Based Questions

### Scenario 1

> Your database has 2 TB of data but queries are slow. Should you immediately increase storage?

**No.**

First investigate:

```text
CPU
Memory
IO
Queries
Indexes
Blocking
Database configuration
```

Storage capacity doesn't automatically solve query performance.

---

### Scenario 2

> Your application is hosted in Azure and the database must not be reachable over the public internet.

What architecture would you investigate?

```text
VNet
  ↓
Private Endpoint
  ↓
Private DNS
  ↓
Azure Database
```

---

### Scenario 3

> A developer can see the Azure SQL resource in the Portal but gets permission denied when querying a table.

What should you investigate?

Separate the two authorization layers:

```text
Azure RBAC
       +
Database permissions
```

The developer may have Azure resource access without having the required database permissions.

---

# 42. Whiteboard Summary

For your teaching session, I recommend ending with this diagram:

```text
                         AZURE DATABASE
                              |
          +-------------------+-------------------+
          |                   |                   |
       COMPUTE             NETWORK             STORAGE
          |                   |                   |
      Processing         Public/Private         Data
      Queries            Firewall               Indexes
      Connections        DNS                    Logs
          |              Private Endpoint
          |
          ↓
     DATABASE ENGINE
          |
          ↓
       DATABASE
          |
     +----+----+----+
     |    |    |    |
  Tables Views Indexes Procedures
```

Then connect it to the Azure hierarchy:

```text
Tenant
  |
Subscription
  |
Resource Group
  |
Database Resource
  |
Database
  |
Tables / Views / Indexes / Procedures
```

---

# 🎯 Lecture 2 Final Takeaway

Students should remember these **10 concepts** before moving forward:

1. **Database** → stores and organizes data.
2. **Database engine** → processes database operations.
3. **Compute** → performs processing.
4. **Storage** → persists data.
5. **Connection** → allows applications/users to communicate with the database.
6. **Authentication** → identifies the user/application.
7. **Authorization** → determines permissions.
8. **Networking** → controls how the database can be reached.
9. **Region/AZ** → determine deployment location and resiliency options.
10. **PaaS vs IaaS** → determines how much infrastructure Azure vs you manage.

---

## Next: Module 1 — Lecture 3

### **Azure SQL Database — First Hands-On Lab**

This is where we can start the practical portion of the course:

**Part 1 — Portal**

```text
Create Resource Group
       ↓
Create Azure SQL Logical Server
       ↓
Create Azure SQL Database
       ↓
Configure Authentication
       ↓
Configure Networking
       ↓
Configure Compute + Storage
       ↓
Connect using Query Editor / SSMS
       ↓
Create Database
       ↓
Create Tables
       ↓
Insert Data
       ↓
Run SQL Queries
```

**Part 2 — Azure CLI**

We'll create the **same complete environment from the command line**, including Resource Group → SQL Server → Database → Firewall → connection testing, so students can see the difference between **Portal administration and CLI administration**.
