# Azure PostgreSQL Backup & Restore — Complete Hands-on

For **Azure Database for PostgreSQL Flexible Server**, it is important to distinguish between **Azure-managed server backups/PITR** and a **logical database backup using pgAdmin**.

---

# 1. Azure Managed Backup & Restore

There are two concepts students should understand:

```text
Azure PostgreSQL Flexible Server
           │
           ├── Azure-managed backup
           │       │
           │       └── Point-in-Time Restore
           │
           └── Logical backup
                   │
                   └── pgAdmin
```

## A. Entire Server Restore — Creates a New Server

The Azure-managed restore process restores the **server** to a new PostgreSQL Flexible Server.

For example:

```text
Production Server
pg-prod
    │
    │ Azure Backup
    │
    ▼
Restore
    │
    ▼
pg-prod-restored
```

The important point is:

> **You don't overwrite the existing PostgreSQL server with the restore operation. Azure creates a new Flexible Server from the selected restore point.**

### Example scenario

Suppose your production server is:

```text
pg-production
```

It contains:

```text
Database
 ├── ecommerce
 ├── inventory
 └── reporting
```

At 10:30 AM, someone accidentally deletes important data.

You can restore the server to a point before the mistake:

```text
pg-production
       │
       │ Point-in-Time Restore
       │
       ▼
pg-production-restore
```

Then:

```text
pg-production-restore
       │
       ├── ecommerce
       ├── inventory
       └── reporting
```

You can validate the restored server before deciding what to do with production.

---

# 2. Point-in-Time Restore — PITR

This is one of the **most important PostgreSQL concepts**.

Imagine:

```text
10:00 ───────── 11:00 ───────── 12:00
                  │
                  │
             Bad operation
                  │
                  ▼
             Data deleted
```

You don't necessarily want yesterday's backup.

You want:

```text
Restore to:
11:55 AM
```

So the process is conceptually:

```text
Azure PostgreSQL
      │
      ├── Backup history
      │
      └── Transaction/WAL information
                │
                ▼
        Selected restore time
                │
                ▼
        New PostgreSQL Server
```

### Example

At:

```text
11:55 AM → Database is healthy
12:00 PM → Developer accidentally deletes orders
12:05 PM → Problem discovered
```

You can perform:

```text
PITR → 11:55 AM
```

and get a new server representing the database state around that point.

---

# 3. Portal — Perform Point-in-Time Restore

Open:

**Azure Portal → PostgreSQL Flexible Servers**

Select your server.

For example:

```text
pg-production
```

Then look for the **Restore** operation.

Depending on the current Azure Portal experience, you may see restore options under the server's management/backup area.

Select the restore option and specify:

### Restore point

Choose:

```text
Point in time
```

Then specify the required date/time.

For example:

```text
September 9, 2026
11:55 AM
```

### New server

Provide a new server name:

```text
pg-production-pitr
```

Then configure the required destination settings and start the restore.

Conceptually:

```text
Source
pg-production
       │
       │
       ▼
Restore Point
11:55 AM
       │
       ▼
Destination
pg-production-pitr
```

---

# 4. What Happens After PITR?

Azure creates a **new Flexible Server**.

You now have:

```text
                 Azure
                   │
          ┌────────┴─────────┐
          │                  │
          ▼                  ▼
   pg-production       pg-production-pitr
       │                    │
       │                    │
   Current data        Data as of
                       restore point
```

Now connect to:

```text
pg-production-pitr
```

and validate:

```sql
SELECT current_database();
```

Check your tables:

```sql
\dt
```

Check the affected table:

```sql
SELECT *
FROM orders;
```

---

# 5. Very Important — PITR Is Not the Same as a pgAdmin Backup

This distinction should be very clear to students.

### Azure PITR

```text
Azure-managed
       ↓
Server-level restore
       ↓
New Flexible Server
       ↓
Point-in-time state
```

### pgAdmin Backup

```text
PostgreSQL
     ↓
pgAdmin
     ↓
Logical backup file
     ↓
Restore database
```

These solve **different problems**.

---

# 6. Install pgAdmin on Your Local Machine

Now let's move to the second method.

**pgAdmin** is a graphical administration tool for PostgreSQL.

You can install it on:

* Windows
* Linux
* macOS

For your class, Windows is probably the easiest demonstration.

Download it from the official PostgreSQL/pgAdmin distribution site:

[pgAdmin official download page](https://www.pgadmin.org/download/?utm_source=chatgpt.com)

Install it using the normal installer.

After installation:

```text
Windows
   ↓
pgAdmin
   ↓
Connect to Azure PostgreSQL
```

---

# 7. Connect pgAdmin to Azure PostgreSQL

Open pgAdmin.

You'll see:

```text
Servers
```

Right-click:

**Servers → Register → Server**

You will get two important tabs:

```text
General
Connection
```

---

## General

Give your server a name:

```text
Azure PostgreSQL
```

---

## Connection

Enter:

### Host

Your Azure PostgreSQL hostname, for example:

```text
pg-day2.postgres.database.azure.com
```

### Port

```text
5432
```

### Maintenance database

```text
postgres
```

### Username

Your PostgreSQL administrator/user.

### Password

Your PostgreSQL password.

Then connect.

---

# 8. Important Networking Requirement

pgAdmin is running on your **local computer**.

Therefore:

```text
Your Laptop
     │
     │ Internet
     ▼
Azure PostgreSQL
```

If the PostgreSQL server uses **public access**, your local machine must be allowed through the PostgreSQL firewall/network rules.

You may need to allow your client IP.

Conceptually:

```text
Laptop Public IP
       │
       ▼
PostgreSQL Firewall
       │
       ▼
Azure PostgreSQL
```

If the PostgreSQL server is configured for **private access only**, your laptop generally cannot connect directly over the public Internet.

You would need appropriate private connectivity, such as VPN/ExpressRoute or another network path into the Azure VNet.

This is a great teaching point:

> **pgAdmin is only the client. Networking and authentication still have to permit the connection.**

---

# 9. Backup a Database Using pgAdmin

Once connected:

```text
Servers
   │
   └── Azure PostgreSQL
          │
          └── Databases
                 │
                 └── ecommerce
```

Right-click:

**ecommerce → Backup...**

You will get the Backup dialog.

---

# 10. Configure Backup

Choose:

### Filename

For example:

```text
C:\PostgreSQLBackup\ecommerce.backup
```

### Format

I recommend demonstrating:

```text
Custom
```

because it works well with PostgreSQL's `pg_dump`/`pg_restore` workflow and allows selective restoration.

You can explain the common formats:

| Format | Description                           |
| ------ | ------------------------------------- |
| Custom | `.backup` / `.dump`, flexible restore |
| Tar    | TAR archive                           |
| Plain  | SQL script                            |

For your first classroom demonstration:

```text
Format = Custom
```

Then click:

**Backup**

---

# 11. What Actually Happens?

pgAdmin is essentially providing a GUI around PostgreSQL backup utilities.

Conceptually:

```text
pgAdmin
   │
   ▼
pg_dump
   │
   ▼
ecommerce
   │
   ▼
ecommerce.backup
```

Your local computer now contains:

```text
C:\PostgreSQLBackup\
       │
       └── ecommerce.backup
```

This is a **logical backup**.

---

# 12. What Does a Logical Backup Contain?

Depending on your backup settings, it can contain things such as:

```text
Database objects
    │
    ├── Tables
    ├── Data
    ├── Indexes
    ├── Constraints
    ├── Functions
    └── Other database objects
```

It is **not** a copy of the Azure PostgreSQL server infrastructure.

For example:

```text
NOT:
Azure Server
 ├── CPU
 ├── RAM
 ├── Networking
 └── Infrastructure
```

Instead:

```text
Database
 ├── Schema
 ├── Tables
 ├── Data
 ├── Functions
 └── Objects
```

---

# 13. Restore the Database Using pgAdmin

Now create a test database.

For example:

```text
ecommerce_restore
```

Then:

```text
Databases
    │
    └── ecommerce_restore
```

Right-click:

**ecommerce_restore → Restore...**

---

# 14. Select Backup File

In the Restore window:

```text
Filename:
C:\PostgreSQLBackup\ecommerce.backup
```

Choose the appropriate format:

```text
Custom or tar
```

Then click:

**Restore**

Conceptually:

```text
ecommerce.backup
       │
       ▼
    pgAdmin
       │
       ▼
 PostgreSQL
       │
       ▼
ecommerce_restore
```

---

# 15. Validate the Restore

After the restore completes:

Expand:

```text
Databases
   ↓
ecommerce_restore
   ↓
Schemas
   ↓
public
   ↓
Tables
```

You should see:

```text
customers
products
orders
order_items
```

Run:

```sql
SELECT *
FROM products;
```

and:

```sql
SELECT count(*)
FROM orders;
```

Compare the restored database with the original.

---

# 16. Azure PITR vs pgAdmin Backup

This is an excellent interview question.

| Feature                     | Azure PITR                          | pgAdmin Backup                   |
| --------------------------- | ----------------------------------- | -------------------------------- |
| Managed by                  | Azure                               | You                              |
| Type                        | Azure-managed backup/restore        | Logical backup                   |
| Scope                       | Server restore                      | Database backup                  |
| Restore                     | New server                          | Database                         |
| Point-in-time               | Yes                                 | Not in the same Azure PITR sense |
| Local backup file           | No                                  | Yes                              |
| Useful for                  | Disaster recovery                   | Migration/export/logical backup  |
| Requires pgAdmin            | No                                  | Yes                              |
| Can move database logically | Limited/indirect                    | Yes                              |
| Infrastructure restored     | Azure creates server                | No                               |
| Best use                    | Accidental deletion/server recovery | Database-level backup/migration  |

---

# 🎯 The Key Concept to Teach

Tell students to remember this simple diagram:

```text
                  PostgreSQL Flexible Server
                            │
             ┌──────────────┴──────────────┐
             │                             │
             ▼                             ▼
      Azure Managed Backup             pgAdmin
             │                             │
             ▼                             ▼
           PITR                        pg_dump
             │                             │
             ▼                             ▼
      New PostgreSQL Server          .backup file
             │                             │
             ▼                             ▼
      Server-level recovery          Database restore
```

---

# 🔥 Real-Time Production Scenario

Suppose your company has:

```text
Production
   │
   ▼
Azure PostgreSQL
   │
   └── ecommerce
```

At 2:00 PM:

```text
Developer accidentally:
DELETE FROM orders;
```

### Option 1 — Azure PITR

Restore the server to:

```text
1:59 PM
```

Azure creates:

```text
pg-production-recovery
```

Validate the data and then plan the production recovery/cutover.

---

### Option 2 — pgAdmin Backup

Suppose you already have:

```text
ecommerce.backup
```

You can restore it into:

```text
ecommerce_restore
```

or another PostgreSQL environment.

This is particularly useful for:

```text
Development
Testing
Migration
Database cloning
Troubleshooting
Data movement
```

---

# ⭐ One More Important Production Point

For a serious production environment, **do not treat pgAdmin logical backups as your only backup strategy**.

A better model is:

```text
                    Production
                        │
             Azure PostgreSQL
                        │
            ┌───────────┴───────────┐
            │                       │
            ▼                       ▼
     Azure-managed backup      Logical backup
            │                       │
            ▼                       ▼
          PITR                 pg_dump/pgAdmin
            │                       │
            ▼                       ▼
      Server recovery         Migration/export
```

**Azure backup/PITR** handles the managed recovery use case, while **logical backups** provide portability and database-level recovery/migration.

### For your 4-hour Day 2 class

I would make this a **single 30–35 minute practical section**:

**10 min:** Azure backup + PITR architecture
**10 min:** Portal PITR → new server
**10 min:** Install/connect pgAdmin + backup
**5 min:** Restore `.backup` → another database and compare

That gives students a very clear understanding of **"Azure restores a server" vs "pgAdmin backs up/restores a database."**
