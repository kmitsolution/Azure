Now we move from **basic PostgreSQL** to the Azure networking architecture you would see in a real production environment.

Our target architecture is:

```text
                    Azure
┌──────────────────────────────────────────────────────────┐
│                                                          │
│                     VNet: 10.0.0.0/16                    │
│                                                          │
│   ┌─────────────────────┐                                │
│   │ Application / VM    │                                │
│   │                     │                                │
│   │  10.0.1.0/24       │                                │
│   └──────────┬──────────┘                                │
│              │                                           │
│              │ Private Network                            │
│              │                                           │
│              ▼                                           │
│   ┌─────────────────────┐                                │
│   │ PostgreSQL Flexible │                                │
│   │ Server              │                                │
│   │                     │                                │
│   │ Private Access      │                                │
│   └─────────────────────┘                                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

We'll understand:

1. VNet
2. Subnet
3. Private Access
4. Private Endpoint
5. Firewall rules
6. NSG
7. DNS
8. Encryption/TLS
9. Public vs private PostgreSQL
10. A complete production example

---

# 1. First: What Problem Are We Solving?

Suppose you have an application running on an Azure VM:

```text
Application
     │
     │ SQL queries
     ▼
PostgreSQL
```

You don't want your database exposed directly to the Internet.

Bad architecture:

```text
             Internet
                 │
                 ▼
        ┌────────────────┐
        │  PostgreSQL    │
        │ Public IP      │
        └────────────────┘
```

Even if a firewall protects it, your database has a public network entry point.

A better architecture is:

```text
                 Internet
                     │
                     X
                     │
              ┌──────┴──────┐
              │     VNet    │
              │             │
              │ VM ────────►│ PostgreSQL
              │             │
              └─────────────┘
```

The database communicates through **private networking**.

---

# 2. What Is an Azure VNet?

A **Virtual Network (VNet)** is your private network inside Azure.

Think of it like your company's internal network.

For example:

```text
VNet
10.0.0.0/16
```

Inside it we create subnets.

```text
VNet: 10.0.0.0/16
│
├── AppSubnet
│   10.0.1.0/24
│
├── DBSubnet
│   10.0.2.0/24
│
└── ManagementSubnet
    10.0.3.0/24
```

You can place your VM in:

```text
10.0.1.0/24
```

and PostgreSQL in the appropriate private networking configuration.

---

# 3. What Is a Subnet?

A subnet is a smaller network inside the VNet.

Example:

```text
VNet
10.0.0.0/16
       │
       ├── AppSubnet
       │   10.0.1.0/24
       │
       └── DBSubnet
           10.0.2.0/24
```

Your VM might receive:

```text
10.0.1.4
```

The database has private network connectivity within the Azure environment.

---

# 4. Private Access

This is an important Azure PostgreSQL concept.

With **private access**, PostgreSQL can be reachable through a private network rather than a public endpoint.

Conceptually:

```text
                Azure VNet
┌─────────────────────────────────────┐
│                                     │
│  VM                                  │
│  10.0.1.4                           │
│      │                              │
│      │ TCP 5432                    │
│      ▼                              │
│  PostgreSQL                         │
│  Private connectivity               │
│                                     │
└─────────────────────────────────────┘

             Internet
                │
                X
        No direct database access
```

This is useful because your application doesn't need to traverse the public Internet to reach the database.

---

# 5. Private Endpoint — What Is It?

This is where many people get confused.

A **Private Endpoint** gives an Azure service a **private IP address inside your VNet**.

For example:

```text
VNet: 10.0.0.0/16

VM
10.0.1.4

Private Endpoint
10.0.2.5
      │
      ▼
Azure Service
PostgreSQL
```

Your application connects to:

```text
10.0.2.5:5432
```

instead of connecting to a public IP.

---

# 6. Private Endpoint vs Private Access

These concepts are related but **not interchangeable**.

### Private Access

For Azure Database for PostgreSQL Flexible Server, private access is the **VNet-based deployment/networking model** for the server.

Conceptually:

```text
PostgreSQL
     │
     └── Integrated with VNet
```

### Private Endpoint

Private Endpoint uses **Azure Private Link** to expose an Azure service through a private IP in your VNet.

Conceptually:

```text
VNet
 │
 └── Private Endpoint
         │
         ▼
     Azure Service
```

So don't memorize:

```text
Private Access = Private Endpoint
```

They are different Azure networking mechanisms.

---

# 7. Firewall Rules

Now let's understand the firewall.

Suppose your PostgreSQL server has public connectivity.

You might configure:

```text
Firewall Rule

Start IP: 49.xx.xx.xx
End IP:   49.xx.xx.xx
```

Meaning:

```text
Your computer
49.xx.xx.xx
     │
     ▼
PostgreSQL Firewall
     │
     ├── 49.xx.xx.xx → ALLOW
     │
     └── Other IPs   → DENY
```

This is **network-level access control**.

---

# 8. Firewall Does NOT Authenticate the User

This is very important.

Suppose the firewall allows:

```text
49.xx.xx.xx
```

That does **not** mean the user is automatically allowed into PostgreSQL.

The connection still needs database authentication.

Think:

```text
                Connection
                    │
                    ▼
             Azure Firewall
                    │
             IP allowed?
                /      \
              NO        YES
              │           │
            BLOCK         ▼
                    PostgreSQL Login
                         │
                    Username/password
                         │
                    Authentication
                         │
                       YES
                         │
                         ▼
                    Database Access
```

So there are multiple security layers.

---

# 9. Firewall Example

Imagine:

```text
PostgreSQL server
postgres-demo.postgres.database.azure.com
```

Firewall:

```text
Rule 1
Name: Office
Start: 49.100.10.20
End:   49.100.10.20
```

Your developer connects:

```text
49.100.10.20
       │
       ▼
Firewall
       │
       ▼
ALLOW
       │
       ▼
PostgreSQL
```

Someone else:

```text
81.200.30.40
       │
       ▼
Firewall
       │
       ▼
DENY
```

Even if they know:

```text
postgres-demo.postgres.database.azure.com
```

they cannot establish the connection from that IP.

---

# 10. NSG — Network Security Group

Now we have another Azure security feature:

**NSG = Network Security Group**

NSGs control network traffic to/from Azure resources associated with supported network interfaces/subnets.

For example:

```text
VM Subnet
10.0.1.0/24
      │
      ▼
     NSG
      │
      ├── Allow SSH from admin IP
      ├── Allow application traffic
      └── Deny unwanted traffic
```

You can think:

```text
Firewall
   ↓
Controls access to PostgreSQL's public endpoint

NSG
   ↓
Controls network traffic around Azure VNet resources
```

But don't assume an NSG automatically controls every managed-service data path. The exact behavior depends on the service's networking model.

---

# 11. Example NSG Rule

Suppose:

```text
VM
10.0.1.10
```

PostgreSQL is privately reachable.

You might have a rule conceptually like:

```text
Source:
10.0.1.0/24

Destination:
PostgreSQL private address

Port:
5432

Protocol:
TCP

Action:
Allow
```

Then:

```text
Application VM
10.0.1.10
      │
      │ TCP 5432
      ▼
Network Security Controls
      │
      ▼
PostgreSQL
```

---

# 12. DNS — The Part People Often Miss

This is extremely important.

When you connect using:

```text
postgres-demo.postgres.database.azure.com
```

your computer doesn't magically know where that server is.

DNS converts:

```text
postgres-demo.postgres.database.azure.com
```

into an IP address.

Conceptually:

```text
Application
     │
     │ DNS query
     ▼
DNS
     │
     ▼
IP address
     │
     ▼
PostgreSQL
```

---

# 13. Public DNS Example

With public access:

```text
postgres-demo.postgres.database.azure.com
                 │
                 ▼
             Public IP
                 │
                 ▼
          PostgreSQL
```

Your application connects using the hostname.

---

# 14. Private DNS Example

With private networking, DNS becomes particularly important.

You want:

```text
postgres-demo.postgres.database.azure.com
```

to resolve appropriately for clients inside your VNet.

Conceptually:

```text
Application
     │
     │
     ▼
Private DNS
     │
     ▼
Private IP
10.0.2.x
     │
     ▼
PostgreSQL
```

So the application doesn't need to hard-code an IP address.

That's why **DNS is a critical part of private networking**.

---

# 15. Why Not Just Use the Private IP?

You might ask:

> Why not put `10.0.2.5` directly in my application configuration?

Because IP addresses can change.

Instead:

```text
Application
     │
     ▼
postgres-demo.postgres.database.azure.com
     │
     ▼
DNS
     │
     ▼
Current private IP
```

Your application remains independent of the underlying IP.

---

# 16. Encrypted Connection

Now let's discuss another very important concept:

**Network security ≠ encryption.**

Suppose your VM connects to PostgreSQL:

```text
VM
 │
 │ TCP 5432
 ▼
PostgreSQL
```

Even if the traffic stays inside Azure's private network, you should still use **TLS/SSL encryption**.

Our previous command used:

```text
sslmode=require
```

For example:

```bash
psql "host=postgres-demo.postgres.database.azure.com port=5432 dbname=companydb user=appuser sslmode=require"
```

The connection is encrypted.

---

# 17. What Does TLS Protect?

Without encryption conceptually:

```text
Application
     │
     │ SQL + credentials + data
     │
     ▼
PostgreSQL
```

With TLS:

```text
Application
     │
     │ 🔒 Encrypted TLS connection
     │
     ▼
PostgreSQL
```

Someone who could observe the network traffic shouldn't be able to simply read:

```text
username
password
SQL query
database data
```

from the encrypted traffic.

---

# 18. Private Network + TLS

The best way to think about these as **different security layers** is:

```text
             Application
                  │
                  │
             ┌────▼────┐
             │   DNS   │
             └────┬────┘
                  │
             Private IP
                  │
             ┌────▼────┐
             │ Network │
             │ Security│
             └────┬────┘
                  │
             TCP 5432
                  │
             ┌────▼────┐
             │  TLS    │
             │Encrypted│
             └────┬────┘
                  │
             PostgreSQL
                  │
             ┌────▼────┐
             │Database │
             │  Auth   │
             └─────────┘
```

Each layer solves a different problem.

---

# 19. Complete Example

Let's imagine an e-commerce application.

```text
              Internet
                  │
                  ▼
           Application
             Gateway
                  │
                  ▼
           Azure Application
              / VM / AKS
                  │
                  │
             Azure VNet
        10.0.0.0/16
                  │
        ┌─────────┴──────────┐
        │                    │
        ▼                    ▼
   App Subnet          PostgreSQL
   10.0.1.0/24         Private Access
        │                    │
        │ TCP 5432           │
        └──────────┬─────────┘
                   │
                   ▼
              PostgreSQL
```

A request comes in:

```text
Customer
   ↓
Application
   ↓
SQL query
   ↓
DNS resolution
   ↓
Private PostgreSQL address
   ↓
Network access controls
   ↓
TLS encrypted connection
   ↓
PostgreSQL authentication
   ↓
Database
```

---

# 20. Public vs Private Architecture

### Development

You might use:

```text
Developer Laptop
       │
       │ Internet
       ▼
PostgreSQL Public Endpoint
       │
       ▼
Firewall
       │
       ▼
PostgreSQL
```

Convenient and easy to learn.

### Production

A typical architecture could look like:

```text
                    Internet
                        │
                        ▼
                  Application
                        │
                        ▼
                 Azure VNet
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
          App Subnet         PostgreSQL
          10.0.1.0/24        Private Access
                                  │
                                  ▼
                             PostgreSQL
```

No need for the database to be directly exposed to the public Internet.

---

# 21. Very Important: Five Different Security Concepts

Memorize this table:

| Feature              | Main purpose                                                        |
| -------------------- | ------------------------------------------------------------------- |
| **VNet**             | Private Azure network                                               |
| **Private Access**   | Private networking model for PostgreSQL Flexible Server             |
| **Private Endpoint** | Private IP access to supported Azure services via Private Link      |
| **Firewall**         | Controls allowed network sources for public PostgreSQL connectivity |
| **NSG**              | Controls network traffic around supported VNet resources            |
| **DNS**              | Resolves hostname to the appropriate IP                             |
| **TLS/SSL**          | Encrypts data in transit                                            |
| **PostgreSQL roles** | Database-level authentication/authorization                         |
| **Azure RBAC**       | Azure resource/control-plane authorization                          |

Notice the last two:

```text
Azure RBAC
     ≠
PostgreSQL permissions
```

We'll come back to this later.

---

# 22. The Security Layers

For your Azure learning, remember this model:

```text
                    Security
                       │
       ┌───────────────┼────────────────┐
       │               │                │
    Network         Encryption       Identity
       │               │                │
       ▼               ▼                ▼
    VNet              TLS          PostgreSQL
    NSG                              Roles
    Firewall                         Entra ID
    Private Access
    Private Endpoint
```

This is much more useful than simply memorizing Azure services.

---

# 23. One Important Correction for Our Earlier Setup

Our earlier server was created using **public access** because that's the easiest way to learn PostgreSQL from your Windows machine.

Don't change it yet.

We'll eventually create a **separate lab environment** for private networking rather than potentially disrupting your working database.

The progression will be:

```text
LAB 1
Windows
   ↓
Public PostgreSQL
   ↓
Firewall
   ↓
TLS
```

Then:

```text
LAB 2
Azure VM
   ↓
VNet
   ↓
Private PostgreSQL
   ↓
DNS
   ↓
TLS
```

Then:

```text
LAB 3
Application
   ↓
Private PostgreSQL
   ↓
Microsoft Entra authentication
   ↓
Azure RBAC
```

---

# 🧪 Practical Exercise — Don't Change Anything Yet

For your **current PostgreSQL server**, go to:

**Azure Portal → PostgreSQL Flexible Server → Networking**

Understand these settings:

```text
Connectivity method
        ↓
Public access

Firewall
        ↓
Your current client IP
```

Then go to:

**Overview → Server name**

Copy the hostname mentally:

```text
xxxxx.postgres.database.azure.com
```

Now compare it with your connection:

```bash
psql "host=xxxxx.postgres.database.azure.com port=5432 dbname=companydb user=appuser sslmode=require"
```

You should now be able to explain:

```text
Why hostname?
        ↓
DNS

Why port 5432?
        ↓
PostgreSQL

Why firewall?
        ↓
Network access control

Why sslmode=require?
        ↓
Encryption in transit

Why appuser?
        ↓
Database identity/authorization
```

---

## Next Lesson

The next practical lesson should be **Azure VM → VNet → PostgreSQL private networking**.

We'll actually build:

```text
VNet: 10.0.0.0/16
       │
       ├── AppSubnet
       │      │
       │      └── Azure VM
       │
       └── PostgreSQL private networking
                │
                └── Private DNS
```

Then we'll SSH/RDP into the VM and prove that:

```text
VM → PostgreSQL
```

works **without using the public database endpoint**.
