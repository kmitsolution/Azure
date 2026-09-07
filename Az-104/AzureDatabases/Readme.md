# Azure Database — Complete Learning Roadmap

### Course Goal

By the end of this course, students should be able to:

* Create and manage Azure databases from the **Azure Portal**
* Create and manage them using **Azure CLI**
* Connect applications to Azure databases
* Configure networking, firewall, private endpoints and DNS
* Implement authentication and security
* Monitor and optimize databases
* Configure backup, HA and disaster recovery
* Migrate on-premises databases to Azure
* Understand when to choose **Azure SQL, MySQL, PostgreSQL, Cosmos DB, etc.**
* Automate database deployments using CLI, ARM/Bicep/Terraform and CI/CD
* Troubleshoot real-world database issues

---

# MODULE 1 — Azure Database Fundamentals

### 1. Introduction to Azure Databases

* What is a database?
* Database vs DBMS
* Relational vs NoSQL
* SQL vs NoSQL
* Database server vs database
* Tables, rows, columns, indexes
* OLTP vs OLAP
* Transaction and ACID
* CAP theorem introduction
* Azure Database services overview

### 2. Azure Database Service Landscape

Understand the major Azure database offerings:

| Category                | Azure Service                    |
| ----------------------- | -------------------------------- |
| Microsoft SQL           | Azure SQL Database               |
| SQL Server              | Azure SQL Managed Instance       |
| SQL Server on VM        | SQL Server on Azure VM           |
| MySQL                   | Azure Database for MySQL         |
| PostgreSQL              | Azure Database for PostgreSQL    |
| NoSQL                   | Azure Cosmos DB                  |
| Key-Value               | Azure Cache for Redis            |
| Distributed NoSQL       | Cosmos DB                        |
| Analytics               | Azure Data Explorer              |
| Data Warehouse          | Microsoft Fabric / Azure Synapse |
| Open-source databases   | MySQL / PostgreSQL               |
| Globally distributed DB | Cosmos DB                        |

We'll also discuss services such as **Azure Database for MariaDB** historically, and why it should not be used for new deployments.

---

# MODULE 2 — Azure SQL Fundamentals

This should be one of the biggest modules because Azure SQL is extremely important for AZ-104/AZ-305 and real-world Azure administration.

### 3. SQL Server vs Azure SQL

Understand:

```text
SQL Server
     |
     +---- SQL Server on Azure VM
     |
     +---- Azure SQL Managed Instance
     |
     +---- Azure SQL Database
```

Compare:

* Management responsibility
* OS access
* SQL Agent
* Backup
* Patching
* Scaling
* Networking
* HA
* Cost

### 4. Create Azure SQL Database — Portal

Step-by-step:

```text
Azure Portal
   ↓
SQL Database
   ↓
Create
   ↓
Subscription
   ↓
Resource Group
   ↓
Database Name
   ↓
Server
   ↓
Compute + Storage
   ↓
Networking
   ↓
Security
   ↓
Review + Create
```

### 5. Create Azure SQL using CLI

For example:

```bash
az group create \
  --name rg-sqldemo \
  --location centralindia
```

Create SQL server:

```bash
az sql server create \
  --name sqlserverdemo123 \
  --resource-group rg-sqldemo \
  --location centralindia \
  --admin-user sqladmin \
  --admin-password "YourPassword"
```

Create database:

```bash
az sql db create \
  --resource-group rg-sqldemo \
  --server sqlserverdemo123 \
  --name EmployeeDB \
  --service-objective S0
```

Then we'll learn how to perform the same operations through the portal.

---

# MODULE 3 — Azure SQL Connectivity

### 6. Connecting to Azure SQL

Learn:

* SSMS
* Azure Data Studio
* SQLCMD
* VS Code
* Application connection strings
* JDBC
* .NET
* Python

Example:

```text
Application
     |
     ↓
Internet / VNet
     |
     ↓
Azure SQL Server
     |
     ↓
EmployeeDB
```

### 7. Firewall

Portal:

```text
SQL Server
 → Networking
 → Firewall rules
```

CLI:

```bash
az sql server firewall-rule create \
  --resource-group rg-sqldemo \
  --server sqlserverdemo123 \
  --name AllowMyIP \
  --start-ip-address X.X.X.X \
  --end-ip-address X.X.X.X
```

Students should understand:

* Client IP
* Azure services
* Public access
* Private access
* Firewall rules
* Server-level firewall

---

# MODULE 4 — Azure SQL Networking

This will be a major hands-on section.

### 8. Public vs Private Connectivity

```text
PUBLIC

Client
  |
Internet
  |
Public IP
  |
Azure SQL
```

versus:

```text
PRIVATE

VM
 |
VNet
 |
Private Endpoint
 |
Azure SQL
```

### 9. Private Endpoint

Learn:

* Private Endpoint
* Private IP
* Private DNS Zone
* DNS resolution
* VNet integration
* Private access
* Disable public network access

### 10. Network Security

Understand:

* Firewall
* NSG
* Private Endpoint
* Service Endpoint
* Private DNS
* Managed Identity
* Microsoft Entra authentication

Important distinction:

> **NSG does not directly control Azure SQL Database traffic the same way it controls traffic to a VM NIC.**

---

# MODULE 5 — Azure SQL Database Administration

### 11. Database Management

* Create database
* Delete database
* Rename database
* Copy database
* Export database
* Import database
* Restore database
* Database users
* Roles
* Permissions

### 12. SQL Authentication

```sql
CREATE USER student1 WITH PASSWORD = 'Password';
```

Understand:

* Server administrator
* SQL authentication
* Database users
* Roles
* Permissions

### 13. Microsoft Entra Authentication

Learn:

```text
User
 ↓
Microsoft Entra ID
 ↓
Azure SQL
 ↓
Database
```

---

# MODULE 6 — Azure SQL Compute & Pricing

### 14. Azure SQL Purchasing Models

Understand:

* DTU
* vCore
* Serverless
* Provisioned compute
* Hyperscale

### 15. Service Tiers

Study:

* General Purpose
* Business Critical
* Hyperscale

Understand:

```text
Workload
   ↓
Performance requirement
   ↓
Compute
   ↓
Storage
   ↓
Service Tier
```

### 16. Scaling

Portal + CLI:

* Scale up
* Scale down
* Change compute
* Storage
* Serverless configuration
* Auto-pause concepts

---

# MODULE 7 — Backup, Restore & HA

### 17. Azure SQL Backup

Understand:

* Automatic backups
* Point-in-time restore
* Long-term retention
* Geo-redundant backup

### 18. Restore

Hands-on:

```text
Database
   ↓
Restore
   ↓
Point in Time
```

CLI exercises will include restore and database copy operations.

### 19. High Availability

Understand:

* Zone redundancy
* Business Critical architecture
* Availability
* Failover concepts

### 20. Disaster Recovery

Learn:

* Geo-replication
* Failover groups
* Regional disaster
* RPO
* RTO

Example:

```text
India Central
    |
 Primary DB
    |
    | Geo Replication
    ↓
India South
    |
 Secondary DB
```

---

# MODULE 8 — Azure SQL Managed Instance

This deserves a separate module.

### 21. What is Azure SQL Managed Instance?

Compare:

```text
SQL Server VM
      ↓
Managed Instance
      ↓
Azure SQL Database
```

### 22. Create Managed Instance

Portal + CLI concepts.

### 23. Managed Instance Networking

Learn:

* Dedicated subnet
* VNet
* NSG
* Route tables
* Private connectivity
* DNS

### 24. SQL MI Administration

* SQL Agent
* Jobs
* Databases
* Logins
* Backup
* Maintenance
* Security

### 25. SQL Database vs SQL MI vs SQL VM

A very important interview topic.

---

# MODULE 9 — SQL Server on Azure VM

### 26. Deploy SQL Server VM

```text
Azure VM
  +
SQL Server
```

Learn:

* Marketplace image
* SQL configuration
* Storage
* Data disk
* TempDB
* Networking
* NSG
* Backup

### 27. SQL Server VM Administration

* SQL Server Agent
* Windows authentication
* SQL authentication
* Backup
* Maintenance
* Performance

### 28. When to use SQL VM?

Case studies:

* Legacy applications
* OS-level requirements
* Third-party SQL tools
* Full SQL Server control

---

# MODULE 10 — Azure Database for MySQL

### 29. MySQL Fundamentals

* MySQL architecture
* Database
* Tables
* Users
* Permissions
* Connections

### 30. Azure Database for MySQL

Portal:

```text
Azure Portal
 → Azure Database for MySQL
 → Create
```

CLI:

```bash
az mysql flexible-server create
```

### 31. MySQL Networking

* Public access
* Private access
* Firewall
* Private DNS
* VNet integration

### 32. MySQL Administration

* Users
* Databases
* Backup
* Restore
* Scaling
* Configuration
* Monitoring

---

# MODULE 11 — Azure Database for PostgreSQL

### 33. PostgreSQL Fundamentals

* PostgreSQL architecture
* Schemas
* Tables
* Roles
* Extensions
* Indexes

### 34. Azure Database for PostgreSQL Flexible Server

Portal + CLI.

Example:

```bash
az postgres flexible-server create
```

### 35. PostgreSQL Networking

* Public access
* Private access
* VNet
* Private DNS
* Firewall

### 36. PostgreSQL Administration

* Users
* Roles
* Extensions
* Backup
* Restore
* Scaling
* Configuration
* Monitoring

---

# MODULE 12 — Azure Cosmos DB

This will be another major module.

### 37. Why NoSQL?

Compare:

```text
SQL

Table
 ├── Row
 ├── Row
 └── Row
```

with:

```text
NoSQL

Container
 ├── JSON document
 ├── JSON document
 └── JSON document
```

### 38. Cosmos DB Architecture

Learn:

* Account
* Database
* Container
* Item
* Partition
* Partition key
* Request Units

### 39. Create Cosmos DB

Portal + CLI.

### 40. Cosmos DB APIs

Important:

* NoSQL API
* MongoDB API
* Cassandra API
* Gremlin API
* Table API

We'll explain **when each API should be used**.

---

# MODULE 13 — Cosmos DB Deep Dive

### 41. Partitioning

One of the most important Cosmos DB concepts.

```text
Container
     |
     +--- Partition A
     |
     +--- Partition B
     |
     +--- Partition C
```

Understand:

* Partition key
* Logical partition
* Physical partition
* Hot partition
* Cardinality

### 42. Request Units — RU/s

Learn:

* What is RU?
* Provisioned throughput
* Autoscale
* Serverless
* RU consumption

### 43. Consistency Levels

Study:

```text
Strong
Bounded Staleness
Session
Consistent Prefix
Eventual
```

### 44. Global Distribution

```text
India
  |
Cosmos DB
  |
USA
  |
Europe
```

Learn:

* Multi-region writes
* Read regions
* Replication
* Failover

---

# MODULE 14 — Other Azure Databases

We'll cover the important services without giving each one the same depth as Azure SQL/MySQL/PostgreSQL.

### 45. Azure Cache for Redis

* Cache vs database
* Redis architecture
* Keys/values
* Cache-aside pattern
* Session storage
* Performance

### 46. Azure Data Explorer

* Kusto Query Language
* Logs
* Telemetry
* Time-series data
* IoT scenarios

### 47. Azure Table Storage

Understand:

```text
Storage Account
   ↓
Table
   ↓
PartitionKey
   ↓
RowKey
   ↓
Entity
```

### 48. Azure Storage Queues

Understand why a queue isn't a traditional database and where it fits in application architecture.

---

# MODULE 15 — Database Security

Very important for real-world Azure administration.

### 49. Authentication

Compare:

* SQL Authentication
* Microsoft Entra Authentication
* Managed Identity
* Service principals

### 50. Authorization

* Users
* Roles
* RBAC
* Database permissions

Understand:

> **Azure RBAC ≠ database permissions**

### 51. Encryption

Learn:

* Encryption at rest
* TDE
* Customer-managed keys
* Encryption in transit
* TLS

### 52. Secrets

Integrate:

```text
Application
     ↓
Managed Identity
     ↓
Key Vault
     ↓
Database
```

---

# MODULE 16 — Monitoring & Troubleshooting

### 53. Azure Monitor

* Metrics
* Logs
* Alerts
* Diagnostic settings

### 54. SQL Monitoring

Understand:

* CPU
* Memory
* Connections
* DTU
* Query performance
* Storage

### 55. Troubleshooting Scenarios

Real-world labs:

> Application cannot connect to database.

Troubleshooting flow:

```text
Application
 ↓
DNS
 ↓
Network
 ↓
Firewall
 ↓
Private Endpoint
 ↓
Authentication
 ↓
Database
```

Other scenarios:

* Login failed
* Timeout
* Database unavailable
* High CPU
* High DTU
* Storage full
* Slow query
* Connection limit

---

# MODULE 17 — Database Performance

### 56. SQL Performance

Learn:

* Indexes
* Query plans
* Missing indexes
* Statistics
* Blocking
* Deadlocks

### 57. Query Optimization

Example:

```sql
SELECT *
FROM Employees
WHERE DepartmentId = 10;
```

Understand how indexing can improve performance.

### 58. Azure SQL Intelligent Performance

* Query Performance Insight
* Automatic tuning
* Recommendations

---

# MODULE 18 — Migration to Azure

This is essential for advanced learners.

### 59. Migration Strategies

Understand:

```text
On-Premises
     |
     +---- Rehost
     |
     +---- Replatform
     |
     +---- Refactor
     |
     +---- Rewrite
```

### 60. SQL Server Migration

Study:

* Azure Database Migration Service
* Backup/restore
* BACPAC
* DACPAC
* Online migration
* Offline migration

### 61. MySQL Migration

```text
On-Prem MySQL
      ↓
Azure MySQL Flexible Server
```

### 62. PostgreSQL Migration

```text
On-Prem PostgreSQL
      ↓
Azure PostgreSQL Flexible Server
```

---

# MODULE 19 — DevOps & Database Automation

This is where we move from administrator → cloud engineer.

### 63. Azure CLI Automation

Create scripts:

```bash
az group create
az sql server create
az sql db create
```

### 64. PowerShell

Database automation using Azure PowerShell.

### 65. Infrastructure as Code

Learn:

```text
ARM
 ↓
Bicep
 ↓
Terraform
```

Example architecture:

```text
Terraform
    |
    +---- VNet
    |
    +---- Private Endpoint
    |
    +---- SQL
    |
    +---- Key Vault
```

### 66. CI/CD Database Deployment

Example:

```text
GitHub / Azure DevOps
        ↓
Build
        ↓
Database Migration
        ↓
Deploy
        ↓
Test
```

---

# MODULE 20 — Application Integration

### 67. .NET + Azure SQL

```text
ASP.NET Core
      ↓
Connection String
      ↓
Azure SQL
```

### 68. Python + PostgreSQL/MySQL

Learn application connectivity.

### 69. Connection Pooling

Why applications shouldn't create a new database connection for every request.

### 70. Managed Identity

Build an application that connects to Azure SQL **without storing a database password**.

---

# MODULE 21 — Advanced Architecture

Now we move into AZ-305-level thinking.

### 71. Database Architecture Patterns

Study:

* Single database
* Database-per-tenant
* Shared database
* Read replicas
* CQRS
* Caching
* Event-driven architecture

### 72. Multi-Region Architecture

Example:

```text
             Traffic Manager
                    |
          +---------+---------+
          |                   |
      Region 1             Region 2
          |                   |
       Azure SQL          Azure SQL
          |                   |
          +------ Geo -------+
```

### 73. Highly Available Application

Final architecture:

```text
                 Users
                   |
              Front Door
                   |
          Application Gateway
                   |
             Load Balancer
                   |
          Application VMs/AKS
                   |
          +--------+--------+
          |                 |
       Redis             Database
                            |
                    Private Endpoint
                            |
                         Azure SQL
```

---

# MODULE 22 — Advanced Security Architecture

### 74. Zero Trust Database Architecture

```text
User
 ↓
Entra ID
 ↓
Application
 ↓
Managed Identity
 ↓
Key Vault
 ↓
Private Endpoint
 ↓
Database
```

### 75. Network Isolation

* No public access
* Private Endpoint
* Private DNS
* Hub-Spoke
* Azure Firewall
* NSG
* Route tables

---

# MODULE 23 — Real-World Projects

I recommend finishing the course with **projects rather than only lectures**.

### Project 1 — Azure SQL Web Application

```text
User
 ↓
Web App
 ↓
Azure SQL
```

### Project 2 — Secure Private Database

```text
VM
 ↓
VNet
 ↓
Private Endpoint
 ↓
Azure SQL
```

### Project 3 — MySQL Application

```text
Linux VM
 ↓
Python Application
 ↓
Azure MySQL
```

### Project 4 — PostgreSQL Application

```text
Application
 ↓
PostgreSQL
```

### Project 5 — Cosmos DB Application

```text
Web API
 ↓
Cosmos DB
 ↓
JSON Documents
```

### Project 6 — Production Architecture

Combine:

```text
Azure Front Door
       ↓
Application
       ↓
Managed Identity
       ↓
Key Vault
       ↓
Private Endpoint
       ↓
Azure SQL
       ↓
Redis Cache
       ↓
Monitoring
```

---

# MODULE 24 — Interview & Certification Preparation

At the end of every major module, we'll have:

### Azure Portal Questions

Example:

> How do you create an Azure SQL Database?

### CLI Questions

> How do you create Azure SQL using Azure CLI?

### Scenario Questions

> Your application cannot connect to Azure SQL. How will you troubleshoot it?

### Architecture Questions

> When would you choose Azure SQL Database instead of SQL Managed Instance?

### AZ-104 Questions

* Database networking
* Security
* Monitoring
* Backup
* RBAC
* Private Endpoint

### AZ-305 Questions

* Database selection
* HA/DR
* Multi-region
* Migration
* Cost optimization
* Architecture decisions

---

# Recommended Course Structure

I would structure the actual YouTube/course lectures approximately like this:

| Phase | Topics                      | Level        |
| ----- | --------------------------- | ------------ |
| 1     | Azure Database Fundamentals | Beginner     |
| 2     | Azure SQL                   | Beginner     |
| 3     | SQL Networking              | Beginner     |
| 4     | SQL Administration          | Intermediate |
| 5     | Backup & HA/DR              | Intermediate |
| 6     | SQL Managed Instance        | Intermediate |
| 7     | SQL Server on Azure VM      | Intermediate |
| 8     | MySQL                       | Intermediate |
| 9     | PostgreSQL                  | Intermediate |
| 10    | Cosmos DB                   | Intermediate |
| 11    | Redis & Other DBs           | Intermediate |
| 12    | Security                    | Advanced     |
| 13    | Monitoring                  | Advanced     |
| 14    | Performance                 | Advanced     |
| 15    | Migration                   | Advanced     |
| 16    | CLI/PowerShell              | Advanced     |
| 17    | Bicep/Terraform             | Advanced     |
| 18    | DevOps                      | Advanced     |
| 19    | Application Integration     | Advanced     |
| 20    | Architecture & Projects     | Expert       |

## The teaching pattern for **every database**

I suggest we follow the same structure throughout the course:

```text
1. What is it?
        ↓
2. Why do we need it?
        ↓
3. When should we use it?
        ↓
4. Architecture
        ↓
5. Pricing / SKUs
        ↓
6. Create using Azure Portal
        ↓
7. Create using Azure CLI
        ↓
8. Connect to it
        ↓
9. Networking
        ↓
10. Security
        ↓
11. Backup
        ↓
12. Restore
        ↓
13. Monitoring
        ↓
14. Performance
        ↓
15. HA/DR
        ↓
16. Migration
        ↓
17. Automation
        ↓
18. Real-world use case
        ↓
19. Troubleshooting
        ↓
20. Interview questions
```

This approach will make the course useful for **AZ-104, AZ-305, Azure Administrator, Azure Developer, DevOps Engineer, and real-world cloud projects**, rather than being just a collection of database tutorials.

**Next step:** We can start with **Module 1, Lecture 1 — “Azure Databases Explained: Complete Database Landscape in Azure”**, and I can give you the full teaching notes, architecture diagrams, Portal steps, CLI commands, practical lab, interview questions, and a hands-on assignment.
