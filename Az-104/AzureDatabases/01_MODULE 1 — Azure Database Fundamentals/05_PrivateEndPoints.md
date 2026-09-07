# Module 1 — Lecture 5

# Azure SQL Networking — Public Access, Firewall, Private Endpoint, VNet & Private DNS

This is a **very important hands-on lecture** because database connectivity problems in Azure are frequently networking problems.

Students should learn to answer:

> **"My application cannot connect to Azure SQL. Where do I start troubleshooting?"**

We'll build that understanding step by step.

---

# 1. Learning Objectives

By the end of this lecture, students will understand:

* Azure SQL public endpoint
* Private endpoint
* Public vs private access
* SQL firewall
* Server-level firewall rules
* Client IP
* VNet
* Subnet
* Private IP
* Private DNS Zone
* Service Endpoint vs Private Endpoint
* NSG and Azure SQL
* Application → Database connectivity
* Basic network troubleshooting
* Portal configuration
* Azure CLI configuration
* Real-world architecture

---

# 2. Start With the Simplest Architecture

From Lecture 3, we created:

```text
Laptop
   |
Internet
   |
Public Endpoint
   |
Azure SQL
```

For example:

```text
sqlserverkmit2026abc.database.windows.net
```

This is the **public connectivity model**.

---

# 3. What Is a Public Endpoint?

A public endpoint allows clients to reach the Azure SQL service through a public network endpoint.

Conceptually:

```text
                  INTERNET
                     |
                     ↓
              Public Endpoint
                     |
                     ↓
                Azure SQL
```

The fact that the endpoint is public does **not** mean everyone automatically has database access.

Network access is controlled by mechanisms such as the Azure SQL firewall and authentication.

---

# 4. Azure SQL Firewall

The firewall controls which public client IP addresses can connect through the public endpoint.

Example:

```text
Internet
   |
   ↓
Azure SQL Firewall
   |
   +---- 203.0.113.10 → ALLOW
   |
   +---- 203.0.113.20 → DENY
   |
   +---- 203.0.113.30 → DENY
```

The firewall is a **network access control layer**.

It does not replace database authentication.

---

# 5. Firewall vs Authentication

This is a very important distinction.

Suppose:

```text
Your IP = Allowed
```

You can reach the endpoint.

But if:

```text
Username = Wrong
Password = Wrong
```

you still cannot authenticate.

Therefore:

```text
Network Access
      +
Authentication
      +
Authorization
```

are separate layers.

---

# 6. Complete Public Connectivity Flow

```text
Laptop
  |
  ↓
DNS
  |
  ↓
Public IP / Endpoint
  |
  ↓
Azure SQL Firewall
  |
  ↓
SQL Authentication / Entra Authentication
  |
  ↓
Database Authorization
  |
  ↓
EmployeeDB
```

This is a great troubleshooting flow to teach.

---

# 7. Portal — Check Firewall

Open:

```text
Azure Portal
   ↓
SQL Servers
   ↓
Your SQL Server
   ↓
Networking
```

Look for:

```text
Public network access
Firewall rules
```

You may see a rule similar to:

```text
AllowMyIP
Start IP: X.X.X.X
End IP:   X.X.X.X
```

---

# 8. Add a Firewall Rule

Click:

```text
Add your client IPv4 address
```

or manually add a rule.

Example:

```text
Rule Name:
OfficeIP

Start IP:
203.0.113.10

End IP:
203.0.113.10
```

Save.

---

# 9. Important Problem — Dynamic Public IP

Suppose yesterday your IP was:

```text
203.0.113.10
```

Today:

```text
203.0.113.50
```

Your Azure SQL firewall still contains:

```text
203.0.113.10
```

Therefore:

```text
New IP
  ↓
Firewall
  ↓
DENIED
```

This is a very common beginner troubleshooting issue.

---

# 10. CLI — Firewall Rules

List rules:

```bash
az sql server firewall-rule list \
  --resource-group <RG> \
  --server <SQL_SERVER> \
  -o table
```

Create a rule:

```bash
az sql server firewall-rule create \
  --resource-group <RG> \
  --server <SQL_SERVER> \
  --name AllowMyIP \
  --start-ip-address <YOUR_PUBLIC_IP> \
  --end-ip-address <YOUR_PUBLIC_IP>
```

Delete:

```bash
az sql server firewall-rule delete \
  --resource-group <RG> \
  --server <SQL_SERVER> \
  --name AllowMyIP
```

---

# 11. Azure Services Firewall Option

Depending on the Azure SQL configuration, you may see an option allowing Azure services/resources to access the server.

This is a broad platform-level access option.

Explain to students:

> Don't enable broad access just because an application is running in Azure.

For production, evaluate the actual network architecture.

---

# 12. Why Public Access Isn't Always Ideal

Consider:

```text
                 INTERNET
                    |
         +----------+----------+
         |          |          |
      Hacker      User      Application
                    |
                    ↓
              Azure SQL
```

Even with firewall restrictions, many organizations prefer not to expose the database through a public endpoint.

Instead:

```text
Application
    |
    ↓
VNet
    |
    ↓
Private Endpoint
    |
    ↓
Azure SQL
```

---

# 13. What Is a Private Endpoint?

A **Private Endpoint** provides a private network interface/IP address in your VNet for accessing a supported Azure service.

Conceptually:

```text
                  Azure VNet
              +----------------+
              |                |
              | Application    |
              |      |         |
              |      ↓         |
              | Private IP     |
              |      |         |
              +------|---------+
                     |
                     ↓
               Private Endpoint
                     |
                     ↓
                 Azure SQL
```

---

# 14. Public IP vs Private IP

### Public

```text
Internet
   |
Public IP
   |
Azure SQL
```

### Private

```text
VNet
 |
Private IP
 |
Private Endpoint
 |
Azure SQL
```

Private endpoint traffic stays on Azure's private networking path rather than requiring a public endpoint for the application-to-database connection.

---

# 15. Important Concept

A private endpoint does **not** mean:

> "Azure SQL itself has been moved into my VNet."

Instead:

> A private endpoint creates a private network interface in your VNet that privately connects to the Azure service.

This distinction is very important for AZ-104 and AZ-305.

---

# 16. Private Endpoint Architecture

Let's create this:

```text
                   Azure VNet
          +--------------------------+
          |                          |
          |      Application VM      |
          |             |            |
          |             ↓            |
          |      Private Endpoint    |
          |        10.0.2.5          |
          |             |            |
          +-------------|------------+
                        |
                        ↓
                  Azure SQL
```

Notice:

```text
10.0.2.5
```

is a private IP associated with the private endpoint network interface.

---

# 17. What Is Private DNS?

Now comes the part that confuses many students.

Our application normally connects to:

```text
sqlserver.database.windows.net
```

If we're using a private endpoint, we want that name to resolve appropriately to a private IP.

That's where **Private DNS Zone** comes in.

Conceptually:

```text
Application
     |
     ↓
DNS Query
     |
     ↓
Private DNS Zone
     |
     ↓
Private IP
     |
     ↓
Private Endpoint
     |
     ↓
Azure SQL
```

---

# 18. Why DNS Is Important

Suppose:

```text
Application
    |
    ↓
sqlserver.database.windows.net
```

If DNS returns the public destination:

```text
Public Endpoint
```

your application may not be using the intended private path.

With the appropriate private DNS configuration:

```text
sqlserver.database.windows.net
          ↓
Private DNS resolution
          ↓
Private Endpoint IP
```

Now the application can use the normal database hostname while resolving it to the private endpoint.

---

# 19. Private DNS Zone for Azure SQL

For Azure SQL private endpoint connectivity, the commonly used private DNS zone is:

```text
privatelink.database.windows.net
```

Azure's private endpoint integration can create/configure the required DNS records and zone relationships.

The exact DNS configuration should be verified against the current Azure service documentation for the service you're deploying.

---

# 20. Complete Private Architecture

```text
                         Azure
                          |
                 +--------+--------+
                 |                 |
              VNet              Azure SQL
                 |                 |
          +------+-----+           |
          |            |           |
       App VM      Private EP -----+
          |            |
          +-----+------+
                |
                ↓
          Private DNS Zone
```

More logically:

```text
Application
    |
    ↓
DNS
    |
    ↓
Private IP
    |
    ↓
Private Endpoint
    |
    ↓
Azure SQL
```

---

# 21. Create a Private Endpoint — Portal

Open:

```text
Azure Portal
 ↓
SQL Server
 ↓
Networking
 ↓
Private access
 ↓
Private endpoint
```

Click:

```text
Create
```

---

# 22. Private Endpoint — Basics

Enter:

```text
Subscription:
Your subscription

Resource Group:
rg-azuredatabase-lab

Name:
pe-sqlserver

Region:
Central India
```

The private endpoint must be deployed into a VNet/subnet that can reach the application.

---

# 23. Select Resource

Choose:

```text
Connection method:
Connect to an Azure resource in my directory
```

Then select:

```text
Resource type:
Microsoft.Sql/servers
```

Select your SQL Server.

The exact portal labels can change, but the underlying concept is:

```text
Private Endpoint
      ↓
Target Azure SQL logical server
```

---

# 24. Networking

Select:

```text
Virtual network:
vnet-database-lab

Subnet:
snet-private-endpoints
```

Example architecture:

```text
vnet-database-lab
       |
       +-- snet-app
       |
       +-- snet-private-endpoints
```

---

# 25. Private DNS Integration

Enable the appropriate private DNS integration.

You may see an option such as:

```text
Integrate with private DNS zone
```

Choose the appropriate private DNS zone.

For Azure SQL, this commonly involves:

```text
privatelink.database.windows.net
```

---

# 26. Review + Create

Click:

```text
Review + create
```

Then:

```text
Create
```

After deployment, you'll have:

```text
VNet
 |
 +-- Private Endpoint
        |
        +-- Private IP
        |
        +-- Private DNS Integration
                    |
                    ↓
                 Azure SQL
```

---

# 27. Verify Private Endpoint

Open:

```text
Azure Portal
 ↓
Private endpoints
 ↓
pe-sqlserver
```

Look at:

```text
Connection state
Private IP address
Network interface
Virtual network
Subnet
```

You should see a private IP such as:

```text
10.0.2.5
```

The exact IP will depend on your subnet/address space.

---

# 28. Private Endpoint Network Interface

This connects nicely with your earlier NIC/NSG lessons.

A private endpoint has a network interface in your VNet.

Conceptually:

```text
Private Endpoint
       |
       ↓
Network Interface
       |
       ↓
Private IP
```

Students should understand that this is **not the same as attaching a normal NIC to an Azure VM**.

The private endpoint's network interface represents the private connection to the Azure service.

---

# 29. Private Endpoint + DNS

Now check:

```text
Private DNS Zone
```

You may see a record corresponding to your SQL server.

Conceptually:

```text
SQL Server Hostname
        |
        ↓
Private DNS
        |
        ↓
10.0.2.5
```

---

# 30. Testing From an Azure VM

For a proper private-endpoint lab, create a Linux VM inside the same VNet or a connected VNet.

Architecture:

```text
                   VNet
                    |
          +---------+---------+
          |                   |
       Linux VM          Private Endpoint
          |                   |
          |                   ↓
          +-------------- Azure SQL
```

From the VM:

```bash
nslookup <SQL_SERVER>.database.windows.net
```

or:

```bash
dig <SQL_SERVER>.database.windows.net
```

The result should resolve according to your private DNS configuration, typically to the private endpoint address.

---

# 31. Test TCP Connectivity

From Linux:

```bash
nc -vz <SQL_SERVER>.database.windows.net 1433
```

Or:

```bash
timeout 5 bash -c '</dev/tcp/<SQL_SERVER>.database.windows.net/1433'
```

From Windows:

```powershell
Test-NetConnection <SQL_SERVER>.database.windows.net -Port 1433
```

---

# 32. Important Troubleshooting Model

If:

```text
nslookup
```

fails:

> Investigate DNS.

If DNS resolves correctly but:

```text
Test-NetConnection
```

fails:

> Investigate network path/firewall/routing/access configuration.

If TCP connectivity succeeds but login fails:

> Investigate authentication and database permissions.

This gives us a powerful troubleshooting sequence:

```text
DNS
 ↓
Network
 ↓
Port
 ↓
Authentication
 ↓
Authorization
```

---

# 33. Service Endpoint vs Private Endpoint

Very important interview topic.

### Service Endpoint

Conceptually:

```text
VNet
 |
Service Endpoint
 |
Azure Service
```

It extends VNet identity to supported Azure services over Azure networking.

### Private Endpoint

Conceptually:

```text
VNet
 |
Private IP
 |
Private Endpoint
 |
Azure Service
```

The private endpoint gives the service a private connectivity path represented by a private IP in your VNet.

---

# 34. Comparison

| Service Endpoint                                            | Private Endpoint                       |
| ----------------------------------------------------------- | -------------------------------------- |
| Uses Azure service endpoint mechanism                       | Uses private IP/NIC                    |
| Service remains accessed through service's endpoint         | Provides private endpoint connectivity |
| Doesn't provide a private IP for the service in your subnet | Private IP exists in your VNet         |
| Simpler architecture                                        | Stronger private-access model          |
| Service-specific support                                    | Service-specific support               |

For modern private database architectures, **Private Endpoint is an important pattern to know well.**

---

# 35. Where Does NSG Fit?

This is where we connect to your previous Azure networking lectures.

Suppose:

```text
VM
 |
NIC
 |
NSG
 |
Subnet
 |
Private Endpoint
 |
Azure SQL
```

NSGs control network traffic associated with supported network interfaces/subnets.

But remember:

> **An NSG is not the Azure SQL firewall.**

They are different security layers.

---

# 36. NSG vs Azure SQL Firewall

### NSG

Controls network traffic associated with Azure network interfaces/subnets.

```text
VM NIC
 ↓
NSG
```

### Azure SQL Firewall

Controls public endpoint access to Azure SQL.

```text
Internet
 ↓
Azure SQL Public Endpoint
 ↓
SQL Firewall
```

### Private Endpoint

Provides private connectivity:

```text
VNet
 ↓
Private Endpoint
 ↓
Azure SQL
```

---

# 37. Production Architecture

Now show students a more realistic design:

```text
                         Internet
                            |
                            ↓
                     Application Gateway
                            |
                            ↓
                       App Subnet
                            |
                    Application VM/AKS
                            |
                            ↓
                   Private Endpoint
                            |
                    Private DNS Zone
                            |
                            ↓
                       Azure SQL
```

Notice:

```text
NO direct public database access
```

This is a much stronger production pattern.

---

# 38. Azure CLI — Create VNet

Let's build a simple lab network.

```bash
az network vnet create \
  --resource-group rg-azuredatabase-lab \
  --name vnet-database-lab \
  --address-prefix 10.10.0.0/16 \
  --subnet-name snet-app \
  --subnet-prefix 10.10.1.0/24
```

---

# 39. Create Private Endpoint Subnet

```bash
az network vnet subnet create \
  --resource-group rg-azuredatabase-lab \
  --vnet-name vnet-database-lab \
  --name snet-private-endpoints \
  --address-prefixes 10.10.2.0/24
```

Depending on current Azure requirements and your configuration, private endpoint network policies may need to be configured appropriately for the subnet.

---

# 40. Create Private Endpoint Using CLI

Conceptually:

```bash
az network private-endpoint create \
  --resource-group rg-azuredatabase-lab \
  --name pe-sqlserver \
  --vnet-name vnet-database-lab \
  --subnet snet-private-endpoints \
  --private-connection-resource-id <SQL_SERVER_RESOURCE_ID> \
  --group-id sqlServer \
  --connection-name sql-private-connection
```

Get the SQL Server resource ID:

```bash
az sql server show \
  --resource-group rg-azuredatabase-lab \
  --name <SQL_SERVER_NAME> \
  --query id \
  -o tsv
```

Use that value for:

```text
<SQL_SERVER_RESOURCE_ID>
```

---

# 41. Verify Private Endpoint

```bash
az network private-endpoint show \
  --resource-group rg-azuredatabase-lab \
  --name pe-sqlserver \
  -o json
```

List private endpoints:

```bash
az network private-endpoint list \
  --resource-group rg-azuredatabase-lab \
  -o table
```

---

# 42. Get Private IP

The private endpoint's NIC contains the private IP.

First find the NIC:

```bash
az network private-endpoint show \
  --resource-group rg-azuredatabase-lab \
  --name pe-sqlserver \
  --query "networkInterfaces[0].id" \
  -o tsv
```

Then inspect the NIC:

```bash
az network nic show \
  --ids <NIC_RESOURCE_ID> \
  --query "ipConfigurations[].{PrivateIP:privateIpAddress,Subnet:subnet.id}" \
  -o table
```

---

# 43. Private DNS Zone

Create the Azure SQL private DNS zone:

```bash
az network private-dns zone create \
  --resource-group rg-azuredatabase-lab \
  --name privatelink.database.windows.net
```

Link it to the VNet:

```bash
az network private-dns link vnet create \
  --resource-group rg-azuredatabase-lab \
  --zone-name privatelink.database.windows.net \
  --name sql-dns-link \
  --virtual-network vnet-database-lab \
  --registration-enabled false
```

---

# 44. DNS Zone Group

A private DNS zone group can associate the private endpoint with the DNS zone.

Conceptually:

```text
Private Endpoint
      |
      ↓
DNS Zone Group
      |
      ↓
Private DNS Zone
```

CLI:

```bash
az network private-endpoint dns-zone-group create \
  --resource-group rg-azuredatabase-lab \
  --endpoint-name pe-sqlserver \
  --name sql-dns-zone-group \
  --private-dns-zone privatelink.database.windows.net \
  --zone-name sql
```

The exact CLI/API behavior can evolve, so verify with:

```bash
az network private-endpoint dns-zone-group --help
```

before running it.

---

# 45. Disable Public Access

Once you've verified private connectivity, you can consider disabling public access.

Portal:

```text
SQL Server
 ↓
Networking
 ↓
Public network access
 ↓
Disable
```

CLI:

```bash
az sql server update \
  --resource-group rg-azuredatabase-lab \
  --name <SQL_SERVER_NAME> \
  --public-network-access Disabled
```

Then your application should use the private path.

---

# 46. Final Private Architecture

After disabling public access:

```text
                         Internet
                            X
                            |
                            X
                            |
                       Azure SQL
                            ↑
                            |
                    Private Endpoint
                            ↑
                            |
                         VNet
                            |
                      Application
```

This is the architecture students should understand.

---

# 47. Very Important: Private Endpoint Doesn't Solve Everything

Suppose we have:

```text
Private Endpoint
```

but DNS isn't configured correctly.

Then:

```text
Application
   |
DNS
   |
Wrong resolution
   |
Connection fails
```

Or:

```text
DNS works
   |
Network routing/access problem
   |
Connection fails
```

Therefore:

> **Private Endpoint + correct DNS + network connectivity are all required for a working private architecture.**

---

# 48. Troubleshooting Scenario

### Problem

Application cannot connect to Azure SQL.

### Step 1 — DNS

```bash
nslookup <SQL_SERVER>.database.windows.net
```

### Step 2 — Check private IP

Verify that DNS resolution matches the intended private endpoint configuration.

### Step 3 — TCP

```bash
nc -vz <SQL_SERVER>.database.windows.net 1433
```

### Step 4 — Authentication

Check:

```text
Username
Password / Token
Authentication method
```

### Step 5 — Authorization

Check:

```text
Database user
Roles
Permissions
```

---

# 49. Hands-On Assignment

## Assignment 3 — Azure SQL Networking

Create:

```text
Resource Group
       |
       ↓
VNet
       |
   +---+----------------+
   |                    |
App Subnet       Private Endpoint Subnet
                        |
                        ↓
                 Private Endpoint
                        |
                        ↓
                   Azure SQL
```

### Tasks

### Task 1

Create:

```text
vnet-database-lab
10.10.0.0/16
```

### Task 2

Create:

```text
snet-app
10.10.1.0/24
```

### Task 3

Create:

```text
snet-private-endpoints
10.10.2.0/24
```

### Task 4

Create a Private Endpoint for your Azure SQL Server.

### Task 5

Create/configure:

```text
privatelink.database.windows.net
```

### Task 6

Create a DNS zone link.

### Task 7

Create an Azure VM in the application subnet.

### Task 8

From the VM run:

```bash
nslookup <SQL_SERVER>.database.windows.net
```

### Task 9

Test port:

```bash
nc -vz <SQL_SERVER>.database.windows.net 1433
```

### Task 10

Verify database connectivity.

### Task 11

Disable public network access on Azure SQL.

### Task 12

Test connectivity again from the VM.

---

# 50. Assignment Questions

Students should answer:

1. What is a public endpoint?
2. What does the Azure SQL firewall do?
3. What is a private endpoint?
4. Does a private endpoint move Azure SQL into your VNet?
5. What is a private IP?
6. Why is DNS required for private endpoint connectivity?
7. What is `privatelink.database.windows.net`?
8. What is the difference between Service Endpoint and Private Endpoint?
9. What is the difference between NSG and Azure SQL Firewall?
10. What happens if DNS works but port 1433 is unreachable?
11. What happens if TCP connectivity works but login fails?
12. Why would you disable public network access after validating private connectivity?

---

# 51. Interview Questions

### Q1. What is an Azure SQL firewall?

A network access control mechanism for Azure SQL's public endpoint that restricts which public IP addresses/ranges can connect.

### Q2. What is a Private Endpoint?

A private endpoint provides a private network interface/IP in a VNet for private connectivity to a supported Azure service.

### Q3. Does Private Endpoint create a private IP for Azure SQL itself?

Conceptually, the private endpoint provides a private IP in your VNet representing the private connection to the Azure service. It does not mean the Azure SQL resource itself has been moved into your VNet.

### Q4. Why do we need Private DNS?

To make database hostnames resolve correctly to the private endpoint from clients using the private network path.

---

# 52. Advanced Interview Question

> **Private Endpoint exists, but the application still connects through the public endpoint. Why?**

Possible issue:

```text
DNS resolution
```

The application may be resolving the database hostname to the public endpoint rather than the private endpoint.

Investigate:

```bash
nslookup <SQL_SERVER>.database.windows.net
```

and inspect:

```text
Private DNS Zone
DNS Zone Link
DNS Zone Group
VNet DNS configuration
```

---

# 53. Another Interview Question

> **The application can resolve the SQL hostname to the private IP, but connection to port 1433 fails. What next?**

Investigate:

```text
Subnet
Network Interface
NSG
Routing
Private Endpoint
Network policies
Azure SQL configuration
```

Then test:

```bash
nc -vz <SQL_SERVER>.database.windows.net 1433
```

---

# 54. Another Scenario

> **The connection reaches Azure SQL but authentication fails. Is this a networking problem?**

Not necessarily.

If TCP connectivity succeeds:

```text
DNS        ✓
Network    ✓
TCP 1433   ✓
```

then investigate:

```text
Authentication
Authorization
Connection string
Database user
```

---

# 55. The Golden Troubleshooting Framework

Make your students memorize this:

```text
          APPLICATION
               |
               ↓
              DNS
               |
               ↓
           IP ADDRESS
               |
               ↓
          NETWORK PATH
               |
               ↓
           TCP / 1433
               |
               ↓
       FIREWALL / ACCESS
               |
               ↓
       AUTHENTICATION
               |
               ↓
       AUTHORIZATION
               |
               ↓
           DATABASE
```

When an application cannot connect:

> **Don't randomly change settings. Walk through the layers.**

---

# 🎯 Lecture 5 Final Summary

We started with:

```text
Laptop
  |
Internet
  |
Public Endpoint
  |
Azure SQL
```

Then learned:

```text
Firewall
```

Then moved to:

```text
VNet
 |
Private Endpoint
 |
Private IP
 |
Private DNS
 |
Azure SQL
```

And finally:

```text
Application
     |
    VNet
     |
Private Endpoint
     |
Private DNS
     |
Azure SQL
```

The major concepts to remember are:

| Concept          | Purpose                                                                  |
| ---------------- | ------------------------------------------------------------------------ |
| Public Endpoint  | Internet-accessible database endpoint                                    |
| Firewall         | Restricts public endpoint access                                         |
| VNet             | Private Azure network                                                    |
| Private Endpoint | Private connectivity to Azure service                                    |
| Private IP       | Private address in VNet                                                  |
| Private DNS      | Resolves database hostname to private endpoint                           |
| NSG              | Network traffic filtering for supported Azure network interfaces/subnets |
| Authentication   | Verifies identity                                                        |
| Authorization    | Determines permissions                                                   |

---

## Next Lecture — Module 1, Lecture 6

### **Azure SQL Authentication & Authorization — SQL Login, Database Users, Microsoft Entra ID, RBAC, Roles & Managed Identity**

We'll take the database we built and answer a very common real-world question:

> **"Should my application use a username/password, Microsoft Entra ID, or Managed Identity to connect to Azure SQL?"**

We'll create users and roles, demonstrate the difference between **Azure RBAC and SQL permissions**, and then build the more secure architecture:

```text
Application
     |
Managed Identity
     |
Microsoft Entra ID
     |
Azure SQL
```

That will take us from **network security → identity security**, which is the natural next step.
