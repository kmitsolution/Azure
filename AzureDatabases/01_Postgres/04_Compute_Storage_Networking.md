# Lesson 4 — Azure PostgreSQL: Compute, Storage & Networking Architecture

The objective is to understand **what Azure is actually providing underneath your PostgreSQL server**.

![Image](https://images.openai.com/static-rsc-4/xx9o0SRijepkGHxmAp820WFOE5JTVugnhPpscikiViQ8JumSodJ3WHx5SAUnZagdlih3Vc9XS6LPOu7zjtiPWV7roeZuzyuZWndlVZx2jxJhFUqb_dy68g_Rbk8pc8XNxb9AOEL_kj-B6w2Bz89UDc4lWfWr6x-VroebNhKGhf6gE4AlzKhatS5nN4iKKcSF?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/7ZRz9agWDJoXVvebHw44AXPJWPwmTFsxWm7Rt7CBKDvR3kTGQpHX4mXaJr80Q260vU8bi6SIwbBeTc3fI-Fu3A3Xf1eO4JAof-oACThA0NQLekVelW2vPa8Fc8GiZq6Tfn9BBhH0hEdfL-2SqKbpFtL0p5ifrxbF-ElNMAK50wSMsPNgX1JgGndoeaFrlar6?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ByMBw2ZT-firadohXGR1SJ9Ta4K77_L6WAtw2HF32b6kq387GhWtHlcuOM4uLHUD4owKS4vxyI_6QC_JP3HsvZhHVVqRspqb5PXU20OQ3Rk4FJl6ZtEsiBzrdV79qU1iGTfcPYNmgYJxtKC2LHq-qOuIMIP8rCidw82jLXygdf50AqzPrRdwLcUjFiTca5fQ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/jgK8DS6KrcuqkvWyNl1F4zhm94cLyItxLogvJPucfHj7r_v7tVnkZaTyEL0LXJ5_9MUJp31KtLWCrGiEA-UxhN7j3TofkBmVIk10j-Ya0T0id88Q9vsasM8r478FFVCeT9XlYMxCDEoGuVwsWW_8NyhYfNuLjq77dAkHglkJcDnd3rrWWuj1gCtC1SXNKClJ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/Xlzsmo26MRaO-tFvcQZM9TngYIQwLl6Y7DI1cqp5d5Hj0DbSVMC0f2MwLSgn6WIHeShr1lWEaA6bO7-KyJzuuedTgCnshkPiZ0VFa5svu1ZLRDEW511CR9IL5ttaqwGKf6OXe8jfESWgzYVuk236IcRhpM2_lmoiAEEiAZBGD8jp6Ze66nHK9md9Yrkeuy3K?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/LBRfaQauVG3hQzDH9A9Yf3ZX5XC9D20Nfn6hyPnMc3DFDopqwnd0PXkMfiOBerbuwoyWPkcyc4V4pI_VB7J9cfTCyLpYIIUruM3fQkNZRE6TG3xNFAYBQQUG3ewB4wnN0ykmlQK4o0lwM0PD7EN_17IdzhkRoLiaBo1ulHd7wgtaY_XfaoVsnfr5o5cb12ks?purpose=fullsize)

---

## 1. Big Picture

When you created:

```text
Azure Database for PostgreSQL
        ↓
Flexible Server
```

Azure is managing several infrastructure components for you.

Think about it like this:

```text
                 Azure
┌──────────────────────────────────────────┐
│                                          │
│     PostgreSQL Flexible Server            │
│                                          │
│   ┌──────────────┐                       │
│   │   Compute    │                       │
│   │              │                       │
│   │ PostgreSQL   │                       │
│   │ Engine       │                       │
│   └──────┬───────┘                       │
│          │                                │
│   ┌──────▼───────┐                        │
│   │   Storage    │                        │
│   │              │                        │
│   │ Data / WAL   │                        │
│   │ Backups      │                        │
│   └──────────────┘                        │
│                                          │
│   ┌──────────────────────────────┐       │
│   │        Networking             │       │
│   │                              │       │
│   │ Public / Private Access      │       │
│   │ Port 5432                    │       │
│   └──────────────────────────────┘       │
│                                          │
└──────────────────────────────────────────┘
```

Let's understand each component.

---

# 2. Compute

**Compute = CPU + RAM used to run PostgreSQL.**

When you created the server, Azure asked you to select something like:

```text
Compute tier
     ↓
Burstable
     ↓
B-series
```

The compute resources run the PostgreSQL database engine.

For example:

```text
PostgreSQL Flexible Server
        │
        └── Compute
             ├── CPU
             ├── RAM
             └── PostgreSQL process
```

### Why does compute matter?

Suppose your application receives:

```text
100 requests/sec
```

and each request executes SQL queries.

PostgreSQL needs CPU and memory to:

* Execute SQL
* Sort data
* Perform joins
* Build query plans
* Maintain connections
* Use database caches

If CPU/RAM is insufficient, database performance can suffer.

---

# 3. Compute Tiers

Azure provides different compute options.

For learning, you might use:

```text
Burstable
```

For production workloads, you may use a more predictable compute configuration.

The important concept is:

```text
More workload
     ↓
More CPU / RAM required
     ↓
Larger compute configuration
```

You can see your current configuration in:

**Azure Portal → PostgreSQL Flexible Server → Compute + storage**

---

# 4. Storage

Now let's separate **Compute** from **Storage**.

Compute runs PostgreSQL.

Storage holds the data.

For example:

```text
Compute
   │
   │ writes
   ▼
Storage
   │
   ├── Tables
   ├── Indexes
   ├── WAL
   └── Database files
```

If you created:

```sql
CREATE TABLE employees (...);
```

and inserted:

```sql
INSERT INTO employees ...
```

the actual persistent database data is stored on Azure-managed storage.

---

# 5. Storage Size

When configuring the server, Azure asks for storage.

For example:

```text
Storage
  ↓
128 GiB
```

The exact minimum/default depends on the current Azure configuration.

The important concept is:

```text
Database size
     ↓
Storage capacity
```

If your database grows:

```text
10 GB
 ↓
50 GB
 ↓
100 GB
 ↓
500 GB
```

your storage requirements grow as well.

---

# 6. Storage vs RAM

This is a very important interview concept.

Don't confuse:

```text
RAM
```

with:

```text
Storage
```

### RAM

Used by PostgreSQL for things like:

```text
Query processing
Caching
Connections
Sorting
```

### Storage

Used for persistent data:

```text
Tables
Indexes
WAL
Database files
```

Simple analogy:

```text
RAM      = Your working desk
Storage  = Your filing cabinet
CPU      = You doing the work
```

---

# 7. Networking

Now the most important Azure-specific area.

Your PostgreSQL server needs a network endpoint so clients can connect.

For example:

```text
Application
     │
     │ TCP 5432
     ▼
PostgreSQL Server
```

PostgreSQL normally listens on:

```text
5432
```

That's why our `psql` connection used:

```text
port=5432
```

---

# 8. Public Access Architecture

In Lesson 2, we used **public access**.

The architecture is:

```text
                Internet
                    │
                    │
              Your Public IP
                    │
                    ▼
          Azure PostgreSQL
          Public Endpoint
                    │
                 TCP 5432
                    │
                    ▼
              PostgreSQL
```

Azure's firewall controls who can connect.

For example:

```text
Client IP
   │
   ▼
Firewall
   │
   ├── Allowed → PostgreSQL
   │
   └── Denied  → Connection blocked
```

That's why we added your current IP address.

---

# 9. Private Access Architecture

For production architectures, you will often see **private networking**.

Instead of exposing PostgreSQL through a public endpoint:

```text
Internet
    │
    X
    │
    └── No direct access
```

you can use private connectivity inside Azure networking.

Conceptually:

```text
                Azure VNet
┌─────────────────────────────────────────┐
│                                         │
│   Application                           │
│   ┌─────────────┐                       │
│   │ Azure VM    │                       │
│   └──────┬──────┘                       │
│          │                              │
│          │ Private Network              │
│          ▼                              │
│   ┌─────────────────┐                   │
│   │ PostgreSQL      │                   │
│   │ Flexible Server │                   │
│   └─────────────────┘                   │
│                                         │
└─────────────────────────────────────────┘
```

This is much closer to what you'll encounter in enterprise Azure environments.

---

# 10. Public vs Private

Remember this table:

| Feature           | Public Access           | Private Access                    |
| ----------------- | ----------------------- | --------------------------------- |
| Internet endpoint | Yes                     | No direct public endpoint         |
| Firewall rules    | Important               | Network controls become important |
| VNet integration  | Not required            | Required                          |
| Development       | Convenient              | Possible                          |
| Production        | Depends on requirements | Common choice                     |

For our learning environment, **public access is easier**.

Later we'll create a private networking setup.

---

# 11. Azure Region

Your PostgreSQL server also exists in a specific Azure **Region**.

For example:

```text
Azure
 │
 └── India
      │
      └── Region
           │
           └── PostgreSQL Flexible Server
```

Your application should ideally be close to your database.

For example:

```text
Application
     │
     │ low latency
     ▼
PostgreSQL
```

rather than:

```text
Application
     │
     │ long distance
     ▼
PostgreSQL
```

This becomes particularly important for application performance.

---

# 12. Availability Zones

Azure regions can contain multiple **Availability Zones**.

Conceptually:

```text
Azure Region
│
├── Availability Zone 1
│
├── Availability Zone 2
│
└── Availability Zone 3
```

For production PostgreSQL deployments, availability architecture becomes important.

You'll later encounter:

```text
High Availability
     ↓
Primary
     +
Standby
```

We'll cover this separately because it deserves its own practical exercise.

---

# 13. Put Everything Together

Now you should visualize Azure PostgreSQL like this:

```text
                         Azure
┌─────────────────────────────────────────────────┐
│                                                 │
│                  Region                         │
│                                                 │
│    ┌──────────────────────────────────────┐     │
│    │ PostgreSQL Flexible Server            │     │
│    │                                      │     │
│    │  ┌───────────────┐                   │     │
│    │  │    Compute    │                   │     │
│    │  │               │                   │     │
│    │  │ CPU + RAM     │                   │     │
│    │  │ PostgreSQL    │                   │     │
│    │  └───────┬───────┘                   │     │
│    │          │                           │     │
│    │          ▼                           │     │
│    │  ┌───────────────┐                   │     │
│    │  │    Storage    │                   │     │
│    │  │               │                   │     │
│    │  │ DB + Indexes  │                   │     │
│    │  │ WAL + Data    │                   │     │
│    │  └───────────────┘                   │     │
│    │                                      │     │
│    │  Networking                          │     │
│    │  Public / Private                   │     │
│    │       │                              │     │
│    └───────┼──────────────────────────────┘     │
│            │                                    │
└────────────┼────────────────────────────────────┘
             │
             │ TCP 5432
             ▼
          Application
```

---

# 🧪 Practical Exercise — Explore Your Server

Don't create anything new yet.

In Azure Portal open:

**PostgreSQL Flexible Server → Compute + storage**

Record:

```text
Compute tier:
VM size:
vCores:
Memory:
Storage:
```

Then open:

**PostgreSQL Flexible Server → Networking**

Record:

```text
Connectivity method:
Public/Private:
Firewall configuration:
```

Then open:

**PostgreSQL Flexible Server → Overview**

Record:

```text
Region:
PostgreSQL version:
Server name:
```

You can paste those values here.

### Next lesson

After you've explored these settings, we'll do a **hands-on Azure architecture exercise**:

**Application/VM → VNet → PostgreSQL → Private Access → NSG/DNS concepts**

That will make the networking architecture much easier to understand before we move into **Azure RBAC + Microsoft Entra authentication**.
