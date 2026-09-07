# Module 1 — Lecture 3

# Azure SQL Database — First Hands-On Lab

Now we move from **concepts → actual Azure implementation**.

In this lecture, we'll create our first Azure SQL Database in two ways:

1. **Azure Portal**
2. **Azure CLI**

We'll then connect to it, create a database table, insert data, query it, and understand what happened behind the scenes.

---

# 1. Learning Objectives

By the end of this lecture, students will be able to:

* Create an Azure Resource Group
* Create an Azure SQL logical server
* Create an Azure SQL Database
* Configure SQL authentication
* Configure networking/firewall
* Connect to Azure SQL
* Create tables
* Insert data
* Run SQL queries
* Delete resources
* Perform the same deployment using Azure CLI
* Understand the relationship between **SQL Server → Database → Tables**

---

# 2. What Are We Going to Build?

Our lab architecture:

```text
                     Your Laptop
                         |
                         |
                    Internet
                         |
                         ↓
                Azure SQL Server
                         |
                         |
                   EmployeeDB
                         |
                +--------+--------+
                |        |        |
             Employees Departments Salaries
```

For today's beginner lab, we'll use **public network access with a firewall rule for your IP**.

Later we'll rebuild the architecture using:

```text
Laptop / VM
     |
    VNet
     |
Private Endpoint
     |
Azure SQL
```

---

# 3. Important Azure SQL Terminology

Before creating anything, explain these four terms.

### Resource Group

Logical container for Azure resources.

```text
Resource Group
     |
     +-- SQL Server
     |
     +-- SQL Database
```

### SQL Logical Server

A management/hosting boundary for Azure SQL databases.

Example:

```text
sqlserverkmit
```

### SQL Database

The actual database.

Example:

```text
EmployeeDB
```

### Tables

Objects inside the database.

```text
EmployeeDB
   |
   +-- Employees
   +-- Departments
   +-- Salaries
```

---

# 4. Our Lab Naming Convention

We'll use:

```text
Resource Group:
rg-azuredatabase-lab

SQL Server:
sqlserver<unique-name>

Database:
EmployeeDB

Firewall Rule:
AllowMyIP
```

⚠️ **Important:** Azure SQL logical server names must be globally unique because the server gets a public DNS name such as:

```text
<server-name>.database.windows.net
```

So don't literally use `sqlserverkmit` if it is already taken.

---

# 5. Part A — Azure Portal

Open:

**Azure Portal → portal.azure.com**

Then search:

```text
Resource groups
```

---

# 6. Create Resource Group

Click:

```text
Create
```

Enter:

```text
Subscription:
Your subscription

Resource group:
rg-azuredatabase-lab

Region:
Central India
```

Then:

```text
Review + create
```

Click:

```text
Create
```

---

# 7. Verify Resource Group

Open:

```text
Resource groups
    ↓
rg-azuredatabase-lab
```

Initially it should be empty.

Soon we'll have:

```text
rg-azuredatabase-lab
       |
       +-- SQL Server
       |
       +-- EmployeeDB
```

---

# 8. Create Azure SQL Database

Search:

```text
SQL databases
```

Click:

```text
Create
```

You'll see the SQL Database creation wizard.

---

# 9. Basics

Select:

```text
Subscription:
Your subscription

Resource group:
rg-azuredatabase-lab
```

Database name:

```text
EmployeeDB
```

---

# 10. Create SQL Server

Under:

```text
Server
```

Click:

```text
Create new
```

Enter a unique server name:

```text
sqlserver<unique-name>
```

Example:

```text
sqlserverkmit2026abc
```

Location:

```text
Central India
```

---

# 11. Authentication

For our first lab, choose SQL authentication.

Example:

```text
Authentication:
SQL authentication

Server admin login:
sqladmin

Password:
<StrongPassword>
```

Use a strong password that satisfies Azure's requirements.

**Do not use a password shown in a tutorial in your real environment.**

---

# 12. What Did We Just Create?

Conceptually:

```text
Azure
 |
Resource Group
 |
 +---- SQL Logical Server
          |
          +---- EmployeeDB
```

The SQL logical server has a DNS endpoint similar to:

```text
sqlserverxxxx.database.windows.net
```

---

# 13. Configure Compute + Storage

For a learning lab, select an inexpensive development-oriented configuration where available.

You may see options such as:

```text
Compute + Storage
```

and different purchasing models/tiers.

For this course, don't focus on optimizing production performance yet.

The important concepts we'll study later are:

* DTU
* vCore
* General Purpose
* Business Critical
* Hyperscale
* Serverless
* Provisioned compute

---

# 14. Networking

This is an important screen.

You'll see options related to:

```text
Connectivity method
```

For today's beginner lab, choose the public-access option that allows selected networks/IP addresses.

Then add your current client IP if the portal offers:

```text
Add current client IP address
```

This creates a firewall rule.

Conceptually:

```text
Your Laptop
     |
Public Internet
     |
     ↓
Azure SQL Public Endpoint
     |
Firewall
     |
     ↓
EmployeeDB
```

---

# 15. Why Is Firewall Required?

Suppose Azure SQL has a public endpoint.

Without network restrictions:

```text
Internet
   |
   +---- Attacker
   |
   +---- Random Client
   |
   +---- Your Laptop
   |
   +---- Application
```

With a firewall rule:

```text
Internet
   |
   ↓
Azure SQL Firewall
   |
   +---- Your IP → ALLOW
   |
   +---- Unknown IP → DENY
```

This is our first introduction to Azure database network security.

---

# 16. Review + Create

Click:

```text
Review + create
```

Azure will validate the configuration.

Then:

```text
Create
```

Wait for deployment.

---

# 17. Verify Deployment

After deployment:

```text
Go to resource
```

You should see:

```text
EmployeeDB
```

And information such as:

```text
Server:
sqlserverxxxx.database.windows.net

Status:
Online
```

---

# 18. Find the SQL Server

Click the server name.

You'll reach the logical SQL server.

Explore:

```text
Overview
Networking
Microsoft Entra ID
Security
Firewall
Databases
```

The exact menu layout may vary as the Azure Portal evolves.

---

# 19. Check Networking

Go to:

```text
SQL Server
   ↓
Networking
```

Look for:

```text
Public network access
Firewall rules
Private endpoint
```

You should see your IP address if you added it during deployment.

---

# 20. Connect to the Database

There are several ways to connect.

For this course we'll eventually cover:

```text
Azure Portal Query Editor
SSMS
Azure Data Studio / VS Code tooling
sqlcmd
Applications
Python
.NET
```

For now, use the easiest available browser-based query experience in the Azure Portal.

---

# 21. Query Editor

Open the database and locate the query/editor option if available.

Authenticate using:

```text
SQL Authentication
```

Enter:

```text
Login:
sqladmin

Password:
YourPassword
```

Once connected, you should be able to execute SQL statements.

---

# 22. Create Your First Table

Run:

```sql
CREATE TABLE Employees
(
    EmployeeId INT PRIMARY KEY,
    EmployeeName VARCHAR(100),
    Department VARCHAR(100),
    Salary DECIMAL(10,2)
);
```

Now:

```text
EmployeeDB
    |
    +-- Employees
```

---

# 23. Insert Data

Run:

```sql
INSERT INTO Employees
(EmployeeId, EmployeeName, Department, Salary)
VALUES
(1, 'Raman', 'IT', 75000),
(2, 'John', 'HR', 65000),
(3, 'David', 'Finance', 80000);
```

---

# 24. Read Data

Run:

```sql
SELECT *
FROM Employees;
```

Expected result:

```text
EmployeeId | EmployeeName | Department | Salary
------------------------------------------------
1          | Raman        | IT         | 75000
2          | John         | HR         | 65000
3          | David        | Finance    | 80000
```

Congratulations — you've just created your first table and inserted data into an Azure SQL Database.

---

# 25. Filter Data

```sql
SELECT *
FROM Employees
WHERE Department = 'IT';
```

---

# 26. Update Data

```sql
UPDATE Employees
SET Salary = 85000
WHERE EmployeeId = 1;
```

Verify:

```sql
SELECT *
FROM Employees
WHERE EmployeeId = 1;
```

---

# 27. Delete Data

```sql
DELETE FROM Employees
WHERE EmployeeId = 3;
```

Verify:

```sql
SELECT *
FROM Employees;
```

---

# 28. Part B — Azure CLI

Now let's perform the same deployment using CLI.

First login:

```bash
az login
```

Check subscription:

```bash
az account show -o table
```

List subscriptions:

```bash
az account list -o table
```

If necessary:

```bash
az account set --subscription "<SUBSCRIPTION_ID>"
```

---

# 29. Create Resource Group

```bash
az group create \
  --name rg-azuredatabase-lab \
  --location centralindia
```

Verify:

```bash
az group show \
  --name rg-azuredatabase-lab \
  -o table
```

---

# 30. Create SQL Logical Server

Use a globally unique name:

```bash
az sql server create \
  --name <UNIQUE_SQL_SERVER_NAME> \
  --resource-group rg-azuredatabase-lab \
  --location centralindia \
  --admin-user sqladmin \
  --admin-password "<STRONG_PASSWORD>"
```

Example:

```bash
az sql server create \
  --name sqlserverkmit2026abc \
  --resource-group rg-azuredatabase-lab \
  --location centralindia \
  --admin-user sqladmin \
  --admin-password "<STRONG_PASSWORD>"
```

---

# 31. Verify SQL Server

```bash
az sql server show \
  --name <SQL_SERVER_NAME> \
  --resource-group rg-azuredatabase-lab \
  -o table
```

---

# 32. Create Database

```bash
az sql db create \
  --resource-group rg-azuredatabase-lab \
  --server <SQL_SERVER_NAME> \
  --name EmployeeDB \
  --service-objective S0
```

The exact available SKU/service objective options can change, so for production deployments we should verify the current supported options rather than blindly reusing an old SKU.

---

# 33. Verify Database

```bash
az sql db show \
  --resource-group rg-azuredatabase-lab \
  --server <SQL_SERVER_NAME> \
  --name EmployeeDB \
  -o table
```

List databases:

```bash
az sql db list \
  --resource-group rg-azuredatabase-lab \
  --server <SQL_SERVER_NAME> \
  -o table
```

---

# 34. Add Firewall Rule

First, determine your current public IP.

On Windows PowerShell:

```powershell
(Invoke-WebRequest -Uri "https://api.ipify.org").Content
```

Suppose it returns:

```text
203.0.113.25
```

Then:

```bash
az sql server firewall-rule create \
  --resource-group rg-azuredatabase-lab \
  --server <SQL_SERVER_NAME> \
  --name AllowMyIP \
  --start-ip-address 203.0.113.25 \
  --end-ip-address 203.0.113.25
```

Verify:

```bash
az sql server firewall-rule list \
  --resource-group rg-azuredatabase-lab \
  --server <SQL_SERVER_NAME> \
  -o table
```

---

# 35. Important Firewall Concept

Notice:

```text
start-ip-address
end-ip-address
```

For a single IP:

```text
Start = 203.0.113.25
End   = 203.0.113.25
```

For an IP range:

```text
Start = 203.0.113.20
End   = 203.0.113.30
```

For production, avoid unnecessarily broad ranges.

---

# 36. Find the Database FQDN

Run:

```bash
az sql server show \
  --resource-group rg-azuredatabase-lab \
  --name <SQL_SERVER_NAME> \
  --query fullyQualifiedDomainName \
  -o tsv
```

You'll get something similar to:

```text
sqlserverkmit2026abc.database.windows.net
```

This is the hostname applications can use to connect to the Azure SQL logical server.

---

# 37. Understanding the Complete CLI Deployment

We have now created:

```text
Azure Subscription
       |
       ↓
Resource Group
       |
       ↓
SQL Logical Server
       |
       ↓
EmployeeDB
       |
       ↓
Firewall Rule
```

This is an excellent diagram to put on your whiteboard.

---

# 38. Portal vs CLI

| Operation      | Portal | CLI                                  |
| -------------- | ------ | ------------------------------------ |
| Resource Group | GUI    | `az group create`                    |
| SQL Server     | GUI    | `az sql server create`               |
| Database       | GUI    | `az sql db create`                   |
| Firewall       | GUI    | `az sql server firewall-rule create` |
| List DBs       | GUI    | `az sql db list`                     |
| Inspect DB     | GUI    | `az sql db show`                     |

This leads us to an important DevOps principle:

> **Anything you can repeatedly configure manually should eventually be considered for automation.**

---

# 39. Lab — Database Operations

Once connected, run these SQL commands.

### Create database object

```sql
CREATE TABLE Departments
(
    DepartmentId INT PRIMARY KEY,
    DepartmentName VARCHAR(100)
);
```

Insert:

```sql
INSERT INTO Departments
VALUES
(1, 'IT'),
(2, 'HR'),
(3, 'Finance');
```

Query:

```sql
SELECT *
FROM Departments;
```

---

# 40. Create Relationship

Now add a foreign key to Employees:

```sql
ALTER TABLE Employees
ADD DepartmentId INT;
```

Update data:

```sql
UPDATE Employees
SET DepartmentId = 1
WHERE EmployeeId = 1;
```

Then add the foreign key:

```sql
ALTER TABLE Employees
ADD CONSTRAINT FK_Employees_Departments
FOREIGN KEY (DepartmentId)
REFERENCES Departments(DepartmentId);
```

Now we have:

```text
Departments
     |
     | DepartmentId
     |
     ↓
Employees
```

This is our first practical example of a **relational database**.

---

# 41. JOIN Query

Run:

```sql
SELECT
    e.EmployeeId,
    e.EmployeeName,
    d.DepartmentName
FROM Employees e
JOIN Departments d
    ON e.DepartmentId = d.DepartmentId;
```

This demonstrates why relational databases are powerful.

---

# 42. What Happens When We Run SELECT?

When we execute:

```sql
SELECT *
FROM Employees;
```

Conceptually:

```text
Your Query
    |
    ↓
Azure SQL
    |
    ↓
SQL Engine
    |
    ↓
Query Processing
    |
    ↓
Storage
    |
    ↓
Result
    |
    ↓
Your Client
```

Later we'll go much deeper into:

* Query optimizer
* Execution plans
* Indexes
* Buffer/cache
* CPU
* IO

---

# 43. Basic Troubleshooting

Suppose you get:

```text
Cannot connect to server
```

Don't immediately assume the database is down.

Use this troubleshooting model:

```text
Client
  |
  ↓
DNS
  |
  ↓
Network
  |
  ↓
Firewall
  |
  ↓
Authentication
  |
  ↓
Database
```

Check each layer.

---

# 44. Troubleshooting — Firewall

If your public IP changed:

```text
Old IP
203.0.113.25
```

but your current IP is:

```text
203.0.113.50
```

Azure SQL firewall may reject your connection.

Solution:

```bash
az sql server firewall-rule update \
  --resource-group rg-azuredatabase-lab \
  --server <SQL_SERVER_NAME> \
  --name AllowMyIP \
  --start-ip-address 203.0.113.50 \
  --end-ip-address 203.0.113.50
```

---

# 45. Troubleshooting — DNS

Test the server hostname.

Windows:

```powershell
nslookup <SQL_SERVER_NAME>.database.windows.net
```

Linux:

```bash
nslookup <SQL_SERVER_NAME>.database.windows.net
```

or:

```bash
dig <SQL_SERVER_NAME>.database.windows.net
```

---

# 46. Troubleshooting — Port

SQL Server normally uses:

```text
TCP 1433
```

From a Windows machine:

```powershell
Test-NetConnection <SQL_SERVER_NAME>.database.windows.net -Port 1433
```

A successful TCP test doesn't prove authentication or database permissions are correct—it only tests connectivity to the endpoint/port.

---

# 47. Security Warning

For this beginner lab we are using:

```text
Public Endpoint
+
Firewall
+
SQL Authentication
```

This is **not automatically the recommended architecture for a production system**.

A production design may instead use:

```text
Application
     |
    VNet
     |
Private Endpoint
     |
Azure SQL
```

and potentially:

```text
Managed Identity
       +
Microsoft Entra ID
       +
Private Endpoint
```

We'll build this architecture later.

---

# 48. Cost Control — VERY IMPORTANT

After completing the lab, don't leave unnecessary resources running.

Check:

```text
Resource Group
      |
      +-- SQL Server
      +-- EmployeeDB
```

When finished with the entire lab, you can delete the resource group:

```bash
az group delete \
  --name rg-azuredatabase-lab \
  --yes
```

This removes the resources contained in that resource group.

⚠️ **Do not run this command if the resource group contains anything you need.**

---

# 49. Assignment 2 — Azure SQL Deployment

## Objective

Create an Azure SQL Database using **both Portal and CLI**.

### Part A — Portal

Create:

```text
Resource Group:
rg-azuredatabase-assignment

SQL Server:
Unique name

Database:
EmployeeDB
```

Configure:

```text
SQL Authentication
Firewall
Compute + Storage
```

---

## Part B — CLI

Create the same environment using:

```bash
az group create
az sql server create
az sql db create
az sql server firewall-rule create
```

---

## Part C — SQL

Create:

```text
Employees
Departments
```

Insert at least:

```text
5 Employees
3 Departments
```

Then perform:

```text
SELECT
INSERT
UPDATE
DELETE
JOIN
```

---

# 50. Assignment Questions

Students must answer:

### Q1

What is the difference between:

```text
SQL Server
SQL Database
Table
```

### Q2

What is the purpose of the Azure SQL logical server?

### Q3

Why is a firewall rule required when using public network access?

### Q4

What is the default SQL Server TCP port?

### Q5

What is the difference between authentication and authorization?

### Q6

What happens if your public IP changes?

### Q7

Why should production databases generally avoid unrestricted public access?

### Q8

What is the purpose of a private endpoint?

### Q9

What is the difference between a Resource Group and a Database?

### Q10

Why would you automate database deployment with CLI instead of manually using the Portal?

---

# 51. Interview Questions

### Beginner

**Q: What is Azure SQL Database?**

A fully managed relational database service based on Microsoft's SQL Server database engine.

**Q: Is Azure SQL Database the same as SQL Server installed on a VM?**

No. Azure SQL Database is a managed PaaS database service, while SQL Server on an Azure VM is an IaaS solution where you manage the VM and SQL Server.

**Q: What is the default SQL Server port?**

TCP 1433 is the standard/default SQL Server port.

**Q: What is a firewall rule?**

A network access rule that determines which client IP addresses/ranges can connect through a public database endpoint.

---

# 52. Scenario Interview Question

> **Your application was working yesterday. Today it cannot connect to Azure SQL. The database is running. What would you check?**

A good answer:

```text
1. DNS resolution
       ↓
2. Client IP
       ↓
3. Firewall
       ↓
4. Network connectivity
       ↓
5. Port 1433
       ↓
6. Username/password/token
       ↓
7. Database permissions
       ↓
8. Application connection string
```

That's the troubleshooting mindset we want throughout this course.

---

# 🎯 Lecture 3 Summary

Today's journey:

```text
Azure Portal
     ↓
Resource Group
     ↓
SQL Logical Server
     ↓
Azure SQL Database
     ↓
Firewall
     ↓
Connect
     ↓
Create Tables
     ↓
Insert Data
     ↓
Query Data
     ↓
CLI Automation
```

The most important architecture to remember is:

```text
                    Azure Subscription
                           |
                           ↓
                    Resource Group
                           |
                           ↓
                  SQL Logical Server
                           |
                 +---------+---------+
                 |                   |
            EmployeeDB          AnotherDB
                 |
          +------+------+ 
          |             |
     Departments     Employees
```

---

## Next Lecture — Module 1, Lecture 4

### **Azure SQL Database — Compute, DTU, vCore, Serverless, Service Tiers & Pricing**

This is where we tackle one of the areas that often confuses beginners:

**"When I create an Azure SQL Database, what are DTU, vCore, General Purpose, Business Critical, Hyperscale and Serverless actually doing?"**

We'll use **real sizing scenarios** and show the **Portal configuration + Azure CLI commands**, including how to scale a database up/down and how to think about **performance vs cost**.
