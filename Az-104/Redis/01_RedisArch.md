Absolutely. Since you want this as **complete course documentation**, let's build it systematically. We'll start with **Module 1 — Redis Architecture**, using the **current Azure Managed Redis** terminology and architecture you are seeing in the Azure Portal.

# Module 1 — Redis Architecture

## Azure Managed Redis — Complete Documentation

### Learning Objectives

By the end of this module, students should understand:

* What Redis is
* Why Redis is extremely fast
* Redis vs PostgreSQL
* Redis architecture
* Key-value model
* Redis data structures
* Cache architecture
* Cache-aside pattern
* Redis nodes
* Redis shards
* Replicas
* Clustering
* High availability
* Azure Managed Redis architecture
* Vertical scaling vs distributed/sharded architecture
* Where Redis fits in an Azure application architecture

---

# 1. What is Redis?

**Redis** is an in-memory data store that provides very low-latency and high-throughput access to data.

Redis is commonly used for:

* Caching
* Session storage
* Counters
* Leaderboards
* Messaging
* Deduplication
* Frequently accessed application data
* Semantic caching
* Some real-time application workloads

Microsoft describes Azure Managed Redis as a managed Redis service based on **Redis Enterprise**. ([Microsoft Learn][1])

---

# 2. Why Do We Need Redis?

Consider a normal application:

```text
                  User
                   │
                   ▼
              Web Application
                   │
                   ▼
              PostgreSQL
                   │
                   ▼
                 Data
```

Suppose 100,000 users request the same product:

```text
100,000 requests
       │
       ▼
Application
       │
       ▼
PostgreSQL
       │
       ├── Query
       ├── Query
       ├── Query
       ├── Query
       └── ...
```

This creates unnecessary database load.

Instead:

```text
                  User
                   │
                   ▼
             Application
                   │
                   ▼
                 Redis
                   │
             Cache Hit?
              /       \
            YES        NO
             │          │
             ▼          ▼
          Return     PostgreSQL
                        │
                        ▼
                      Redis
                        │
                        ▼
                     Return
```

Now most frequently requested data can be served from memory.

---

# 3. Redis vs PostgreSQL

This distinction is extremely important.

## PostgreSQL

PostgreSQL is normally the **system of record**.

```text
PostgreSQL
     │
     ├── Customers
     ├── Products
     ├── Orders
     ├── Payments
     └── Inventory
```

Data is intended to be durable and relational.

---

## Redis

Redis is commonly used as a **fast data layer**.

```text
Redis
   │
   ├── Cached Products
   ├── Sessions
   ├── Temporary Data
   ├── Counters
   └── Frequently Used Data
```

Therefore:

> **Redis normally complements your database; it does not simply replace your PostgreSQL database.**

A Microsoft architecture example explicitly uses Azure Managed Redis as a distributed cache while keeping the relational database as the authoritative system of record. ([Microsoft Learn][2])

---

# 4. Basic Redis Architecture

At the simplest level:

```text
              Application
                   │
                   │ Redis API
                   ▼
            ┌───────────────┐
            │     Redis     │
            │               │
            │ Key → Value   │
            └───────────────┘
```

Example:

```text
Key:

user:100

Value:

John Doe
```

You already demonstrated this in your Node.js application:

```javascript
await client.set("user:100", "John Doe");
```

Retrieve:

```javascript
const value = await client.get("user:100");
```

Result:

```text
John Doe
```

---

# 5. Redis Key-Value Model

Redis fundamentally works with **keys** and associated values.

```text
Key                    Value
────────────────────────────────
user:100               John Doe
product:101            Laptop
product:102            Mobile
session:abc123         ...
```

Think of Redis as:

```text
              Redis
                │
       ┌────────┼─────────┐
       │        │         │
       ▼        ▼         ▼
    user:100 product:101 session:1
       │        │         │
       ▼        ▼         ▼
    John Doe   Laptop   Session data
```

---

# 6. Redis Is More Than Simple Key-Value Storage

Redis supports multiple data structures.

## String

```redis
SET user:100 "John Doe"
GET user:100
```

---

## Hash

Useful for representing an object:

```redis
HSET user:100 name "John Doe" city "Delhi"
```

Conceptually:

```text
user:100
   │
   ├── name = John Doe
   └── city = Delhi
```

---

## List

Useful for queues or ordered collections.

```redis
LPUSH jobs job1
LPUSH jobs job2
```

---

## Set

Useful when you need unique values.

```redis
SADD skills Azure
SADD skills Docker
SADD skills Kubernetes
```

---

## Sorted Set

Useful for rankings and leaderboards.

```text
Player       Score
──────────────────
Raman        950
Amit         900
John         850
```

---

# 7. Redis Memory Architecture

The key reason Redis is fast is that the primary working data is kept in memory.

Conceptually:

```text
              Redis
                │
                ▼
              RAM
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
      Keys    Values   Data structures
```

Compare that with a traditional database:

```text
Application
     │
     ▼
Database
     │
     ▼
Storage
     │
     ▼
Disk / SSD
```

Redis is designed for very fast access to in-memory data, which is why it is commonly used as a low-latency cache.

---

# 8. Cache Architecture

Now introduce the most important real-world Redis pattern.

```text
                     Application
                          │
                          ▼
                       Redis
                          │
                     Cache Hit?
                    /           \
                  YES            NO
                   │              │
                   ▼              ▼
                Return       PostgreSQL
                                  │
                                  ▼
                                Data
                                  │
                                  ▼
                                Redis
                                  │
                                  ▼
                               Return
```

This is commonly called the **Cache-Aside Pattern**.

Microsoft's Azure Managed Redis documentation identifies cache-aside as a common application pattern. ([Microsoft Learn][1])

---

# 9. Cache Hit

Suppose:

```text
product:101
```

is already in Redis.

Application:

```text
GET product:101
```

Redis responds:

```text
Laptop
```

This is a:

> **Cache Hit**

```text
Application
     │
     ▼
   Redis
     │
     ▼
   Found
     │
     ▼
 Return data
```

No PostgreSQL query is required.

---

# 10. Cache Miss

Suppose Redis doesn't contain:

```text
product:999
```

Application:

```text
GET product:999
```

Redis returns:

```text
NULL
```

This is a:

> **Cache Miss**

Application then goes to PostgreSQL:

```text
Application
     │
     ▼
   Redis
     │
     ▼
   MISS
     │
     ▼
 PostgreSQL
     │
     ▼
   Data
```

Then the application can populate Redis:

```text
PostgreSQL
     │
     ▼
Application
     │
     ▼
Redis
```

---

# 11. TTL — Time to Live

Redis allows us to specify how long a key should remain.

Your example:

```javascript
await client.set(
    "user:100",
    "John Doe",
    { EX: 60 }
);
```

means:

```text
Key: user:100
Value: John Doe
TTL: 60 seconds
```

Architecture:

```text
SET
 │
 ▼
Redis
 │
 ├── 60 seconds
 │
 ├── 59 seconds
 │
 ├── 58 seconds
 │
 └── 0
      │
      ▼
    Expire
```

This is extremely useful for cached data.

---

# 12. Why TTL Is Important

Suppose PostgreSQL contains:

```text
Product Price = ₹50,000
```

Redis contains:

```text
Product Price = ₹50,000
```

Later PostgreSQL changes:

```text
Product Price = ₹48,000
```

If Redis contains an old value indefinitely, users might receive stale data.

TTL helps:

```text
Redis
  │
  ▼
Cached data
  │
  ▼
Expires
  │
  ▼
Application retrieves latest data
```

But TTL alone is not a complete cache-consistency strategy; applications often also explicitly invalidate or update cache entries when underlying data changes.

---

# 13. Redis in an Azure Application

A typical Azure architecture might be:

```text
                         Internet
                            │
                            ▼
                     Azure Application
                            │
                   ┌────────┴────────┐
                   │                 │
                   ▼                 ▼
            Azure Managed Redis   PostgreSQL
                   │                 │
                   │                 │
              Fast cache        Source of Truth
```

For example:

### Redis

```text
product:101
product:102
product:103
```

### PostgreSQL

```text
products
customers
orders
payments
inventory
```

---

# 14. Azure Managed Redis Architecture

Now move from **Redis architecture** to **Azure Managed Redis architecture**.

Azure Managed Redis runs on the **Redis Enterprise** stack. Its architecture differs from the older Azure Cache for Redis community-edition-based architecture. ([Microsoft Learn][3])

A simplified view:

```text
                 Azure Managed Redis
                        │
            ┌───────────┴───────────┐
            │                       │
          Node 1                  Node 2
            │                       │
       ┌────┴────┐             ┌────┴────┐
       │         │             │         │
     Shard     Shard         Shard     Shard
```

The actual number of shards depends on the selected SKU/configuration.

---

# 15. What Is a Node?

A **node** is an underlying compute instance participating in the Redis deployment.

Conceptually:

```text
Azure Managed Redis
        │
   ┌────┴────┐
   ▼         ▼
Node 1     Node 2
```

Nodes provide the infrastructure on which Redis processes/shards run.

---

# 16. What Is a Shard?

This is a very important concept.

A **shard** is a Redis server process that handles part of the data/workload.

Conceptually:

```text
Redis
 │
 ├── Shard 1
 ├── Shard 2
 ├── Shard 3
 └── Shard 4
```

Data can be distributed across shards.

Microsoft explains that Azure Managed Redis uses multiple Redis server processes called **shards**, allowing operations to execute in parallel. ([Microsoft Learn][3])

---

# 17. Important Correction About Scaling

This is particularly relevant because you just looked at the **Scale** blade in your Azure Portal.

You might think:

```text
Scale
   ↓
Add Shard
   ↓
Add Node
```

But Azure Managed Redis does **not** give you a simple manual "number of shards" slider.

Microsoft states:

> You can't manually change the number of shards.

The SKU determines the shard configuration, and Azure handles the underlying distribution. ([Microsoft Learn][3])

Therefore, when you use the portal's Scale option:

```text
Current plan
     ↓
Target cache size
     ↓
Target performance
```

you are selecting a different service capacity/performance configuration.

This explains exactly what you saw in your portal.

---

# 18. How the Application Sees Redis

This is another important concept.

Even though internally Redis may have:

```text
Node 1
 ├── Shard 1
 └── Shard 2

Node 2
 ├── Shard 3
 └── Shard 4
```

your application doesn't normally have to manage those individual shards.

The application sees:

```text
             Application
                  │
                  ▼
          Azure Managed Redis
                  │
           Routing handled
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
    Shard 1    Shard 2    Shard 3
```

Azure Managed Redis uses proxy processes to handle connection management and routing between the application and Redis processes. ([Microsoft Learn][3])

---

# 19. High Availability Architecture

For production, availability becomes important.

With HA enabled, Azure Managed Redis distributes primary and replica shards across at least two nodes. ([Microsoft Learn][1])

Conceptually:

```text
                   Azure Managed Redis
                           │
              ┌────────────┴────────────┐
              │                         │
           Node 1                    Node 2
              │                         │
         ┌────┴────┐               ┌────┴────┐
         │         │               │         │
      Primary   Replica         Replica   Primary
```

The exact physical layout is managed by Azure.

---

# 20. Why Replicas?

Suppose:

```text
Primary
   │
   ▼
Failure
```

Without redundancy:

```text
Application
     │
     X
   Redis
```

With HA:

```text
Application
     │
     ▼
Redis
     │
 Primary
     X
     │
 Replica
     │
     ▼
 Continue service
```

The purpose is **availability and resilience**, not simply increasing cache capacity.

---

# 21. Scaling vs High Availability

Students frequently confuse these.

## Scaling

```text
More capacity
       ↓
More workload
```

## High Availability

```text
Failure
   ↓
Redundant resources
   ↓
Service continues
```

Therefore:

```text
Scaling       → Capacity / Performance

High
Availability  → Resilience
```

---

# 22. Redis Cluster Architecture

A simplified conceptual architecture:

```text
                       Application
                            │
                            ▼
                  Azure Managed Redis
                            │
                      Proxy / Routing
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
       Shard 1           Shard 2           Shard 3
          │                 │                 │
       Data A             Data B             Data C
```

Azure Managed Redis internally uses clustering across its SKUs, although smaller instances can have a single shard. ([Microsoft Learn][3])

---

# 23. Redis Cluster Policies

Azure Managed Redis supports three clustering policies:

```text
OSS
Enterprise
Non-clustered
```

Microsoft recommends **OSS clustering for most applications** because it supports higher maximum throughput, while non-clustered configurations are intended for applications that cannot support the clustered protocols. ([Microsoft Learn][3])

For your beginner class, don't go too deeply into protocol differences yet.

Just remember:

```text
Cluster Policy
      │
      ├── OSS
      ├── Enterprise
      └── Non-clustered
```

We'll cover this later in **Redis Configuration**.

---

# 24. Azure Managed Redis Performance Tiers

The current Azure Managed Redis offering includes several performance tiers.

The three in-memory tiers are:

```text
Memory Optimized
Balanced
Compute Optimized
```

There is also:

```text
Flash Optimized
```

which uses both RAM and NVMe Flash storage. ([Microsoft Learn][4])

### Simplified comparison

```text
Memory Optimized
        │
        ▼
More memory relative to CPU

Balanced
        │
        ▼
Balance of memory + CPU

Compute Optimized
        │
        ▼
More CPU relative to memory

Flash Optimized
        │
        ▼
RAM + NVMe Flash
```

For example, the screenshot you showed earlier had:

```text
Balanced
vCPUs: 2
Cache: 0.5 GB
SKU: B0
```

and you selected:

```text
Balanced
vCPUs: 2
Cache: 1 GB
SKU: B1
```

That's a change in service capacity.

---

# 25. Azure Managed Redis Networking

For a production architecture, you don't want application traffic unnecessarily exposed to the public Internet.

Conceptually:

```text
                  Azure VNet
                      │
          ┌───────────┴───────────┐
          │                       │
     Application              Private Endpoint
          │                       │
          │                       ▼
          └──────────────► Azure Managed Redis
```

Azure Managed Redis supports private endpoints for network isolation. ([Microsoft Learn][5])

We'll cover networking separately.

---

# 26. Authentication

Current Azure Managed Redis supports **Microsoft Entra ID authentication**, and new caches have managed identity enabled by default according to Microsoft's current documentation. ([Microsoft Learn][5])

Conceptually:

```text
Application
     │
     ▼
Managed Identity
     │
     ▼
Microsoft Entra ID
     │
     ▼
Azure Managed Redis
```

This can reduce the need to store Redis passwords/keys in application configuration.

---

# 27. Data Persistence

Redis is primarily an in-memory data store, but Azure Managed Redis provides **data persistence** capabilities.

Conceptually:

```text
             Redis
              │
          ┌───┴────┐
          │        │
         RAM     Persistence
                  │
             ┌────┴────┐
             │         │
            RDB       AOF
```

Persistence is important for workloads where Redis data needs additional durability.

However:

> **Persistence is not the same thing as a traditional backup/PITR strategy.**

For backup/export and restore, Azure Managed Redis provides **Import/Export** capabilities. We'll cover this in the Backup & Restore section.

---

# 28. Redis Data Flow — Complete Example

Let's take your e-commerce example.

### Step 1 — User requests product

```text
GET /products/101
```

### Step 2 — Application checks Redis

```text
GET product:101
```

### Step 3 — Cache hit

```text
Redis
 │
 └── product:101
        │
        ▼
      Laptop
```

Return immediately.

---

### Cache miss

```text
Application
     │
     ▼
Redis
     │
     X
   MISS
     │
     ▼
PostgreSQL
     │
     ▼
Product
     │
     ▼
Redis
     │
     ▼
Application
```

This reduces repeated PostgreSQL reads.

---

# 29. Production Architecture

Here's the architecture I recommend students remember:

```text
                         Internet
                            │
                            ▼
                     Azure Application
                            │
                    ┌───────┴────────┐
                    │                │
                    ▼                ▼
             Azure Managed       PostgreSQL
                Redis             Flexible
                    │               Server
                    │                 │
                    │                 ▼
                    │             Persistent
                    │                Data
                    │
                    ▼
              Cached Data
```

More detailed:

```text
                     ┌─────────────────┐
                     │   Application   │
                     └────────┬────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
             Azure Managed         PostgreSQL
                 Redis              Flexible
                    │                 Server
                    │                   │
            ┌───────┴──────┐           │
            │              │           │
         Primary         Replica       │
         Shards          Shards        │
            │              │           │
            └──────────────┘           │
                                       ▼
                                  Source of Truth
```

---

# 30. What Should Go Into Redis?

Good candidates:

```text
Frequently accessed products
User sessions
API responses
Configuration
Counters
Leaderboards
Temporary data
Computed results
```

Examples:

```text
product:101
session:abc123
cart:user:100
counter:pageviews
leaderboard:2026
```

---

# 31. What Should Usually Stay in PostgreSQL?

Don't use Redis as the only location for critical durable data such as:

```text
Orders
Payments
Financial transactions
Customer master data
Inventory source of truth
Audit records
```

Instead:

```text
PostgreSQL
     │
     ├── Orders
     ├── Payments
     ├── Customers
     └── Inventory
     
Redis
     │
     ├── Cached Orders
     ├── Cached Products
     ├── Sessions
     └── Temporary state
```

---

# 32. Important Interview Questions

### Q1. What is Redis?

An in-memory data store designed for very low latency and high throughput.

### Q2. Why is Redis fast?

Primarily because frequently accessed data is served from memory and Redis is optimized for fast data operations.

### Q3. Is Redis a database?

Redis is a data store and can be used for more than caching. In Azure architectures, it is frequently used as a cache, session store, or messaging/data-processing component.

### Q4. Redis vs PostgreSQL?

```text
Redis      → Fast data layer/cache
PostgreSQL → Persistent relational system of record
```

### Q5. What is cache-aside?

The application checks Redis first. If the data isn't there, it gets the data from the database and populates Redis.

### Q6. What is a cache hit?

Requested data is found in Redis.

### Q7. What is a cache miss?

Requested data isn't found in Redis, so the application retrieves it from the source database.

### Q8. What is TTL?

Time To Live — the period after which a key expires.

### Q9. What is a Redis shard?

A Redis server process responsible for part of the data/workload.

### Q10. Can I manually select the number of shards in Azure Managed Redis?

**No.** Azure determines the shard configuration based on the SKU/configuration; Microsoft states that the number of shards cannot be manually changed. ([Microsoft Learn][3])

### Q11. Is HA the same as scaling?

No.

```text
HA       → availability
Scaling  → capacity/performance
```

### Q12. Does Redis replace PostgreSQL?

Normally no. Redis commonly acts as a high-speed data layer while PostgreSQL remains the durable source of truth.

---

# 33. Module 1 — Hands-on Lab

Use the Redis instance you already created.

### Step 1 — Connect from Node.js

```javascript
import { createClient } from "redis";
import "dotenv/config";

const client = createClient({
    url: process.env.REDIS_URL
});

client.on("error", err =>
    console.error("Redis Client Error", err)
);

await client.connect();

console.log("Connected");
```

### Step 2 — Create data

```javascript
await client.set(
    "user:100",
    "John Doe",
    { EX: 60 }
);
```

### Step 3 — Read

```javascript
const value = await client.get("user:100");

console.log(value);
```

### Step 4 — Check TTL

```javascript
const ttl = await client.ttl("user:100");

console.log(`TTL: ${ttl} seconds`);
```

### Step 5 — Check existence

```javascript
const exists = await client.exists("user:100");

console.log(`Exists: ${exists === 1}`);
```

### Step 6 — Delete

```javascript
await client.del("user:100");
```

Then:

```javascript
console.log(await client.get("user:100"));
```

Expected:

```text
null
```

---

# 34. Module 1 Assignment

Ask students to build this:

```text
Redis
 │
 ├── user:100
 ├── user:101
 ├── product:101
 ├── product:102
 └── product:103
```

For each key, demonstrate:

```text
SET
GET
EXISTS
TTL
DEL
```

Then answer:

1. Why is Redis faster than querying PostgreSQL for cached data?
2. What is a cache hit?
3. What is a cache miss?
4. What is TTL?
5. What is a shard?
6. What is a Redis node?
7. What is HA?
8. Why do we need PostgreSQL if we have Redis?
9. What happens when a cached key expires?
10. What happens if Redis is unavailable?

---

# 🧠 Final Architecture to Remember

The single diagram I would put at the end of your **Redis Architecture lecture** is:

```text
                           USERS
                             │
                             ▼
                    ┌─────────────────┐
                    │   APPLICATION   │
                    └────────┬────────┘
                             │
                  ┌──────────┴──────────┐
                  │                     │
                  ▼                     ▼
        ┌──────────────────┐   ┌────────────────────┐
        │ Azure Managed    │   │ PostgreSQL         │
        │ Redis            │   │ Flexible Server    │
        │                  │   │                    │
        │ Cache            │   │ Source of Truth    │
        │ Sessions         │   │ Customers          │
        │ Counters         │   │ Orders             │
        │ Temp Data        │   │ Payments           │
        └────────┬─────────┘   └────────────────────┘
                 │
        ┌────────┴────────┐
        │                 │
     Node 1             Node 2
        │                 │
     Shards              Shards
        │                 │
        └────────┬────────┘
                 │
            HA / Replication
```

### The 5 concepts students must leave with

**1. Redis = fast data layer**

**2. PostgreSQL = persistent source of truth**

**3. Cache-aside = most important caching pattern**

**4. Azure Managed Redis internally uses nodes, shards, routing, and replicas**

**5. Scaling and HA are different concepts**

This architecture is aligned with the **current Azure Managed Redis service**, rather than the older Azure Cache for Redis model. Microsoft's current documentation explicitly identifies Azure Managed Redis as the newer Redis offering and describes its Redis Enterprise-based architecture. ([Microsoft Learn][3])

### Next section of the documentation

**Module 2 — Azure Managed Redis Provisioning & Configuration** should cover, in this order:

`Create Redis → Resource Group → Region → Performance Plan/SKU → Networking → Authentication → Advanced Settings → Cluster Policy → HA → Data Persistence → Encryption → Connect from Node.js → Redis CLI → Monitoring`.

[1]: https://learn.microsoft.com/tr-tr/azure/redis/overview?utm_source=chatgpt.com "What is Azure Managed Redis? - Azure Managed Redis | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/azure/architecture/databases/architecture/write-through-caching-azure-sql-managed-redis?utm_source=chatgpt.com "Write-Through Caching with Azure Managed Redis and Azure SQL Database - Azure Architecture Center | Microsoft Learn"
[3]: https://learn.microsoft.com/sv-se/azure/redis/architecture?utm_source=chatgpt.com "Azure Managed Redis Architecture - Azure Managed Redis | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/azure/redis/whats-new?utm_source=chatgpt.com "What's new in Azure Managed Redis - Azure Managed Redis | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/azure/architecture/guide/multitenant/service/managed-redis?utm_source=chatgpt.com "Azure Managed Redis Considerations for Multitenancy - Azure Architecture Center | Microsoft Learn"
