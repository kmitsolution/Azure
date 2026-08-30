# Lesson 2 — Configure Firewall & Connect to Azure PostgreSQL

In Lesson 1, we created the **Azure Database for PostgreSQL Flexible Server**.

Now we'll connect to it from your **Windows machine**.

Our flow:

```text
Windows Laptop
     |
     | PostgreSQL TCP 5432
     ↓
Azure PostgreSQL Flexible Server
     |
     ↓
PostgreSQL Database
```

## Step 1 — Get your PostgreSQL server name

Azure Portal → **PostgreSQL Flexible Server** → **Overview**

Copy the **Server name**.

It will look like:

```text
postgres-demo-xxxx.postgres.database.azure.com
```

Keep it ready.

---

# Step 2 — Find your public IP address

On your Windows machine, open PowerShell:

```powershell
curl.exe -4 https://ifconfig.me
```

You should get something like:

```text
49.xx.xx.xx
```

This is the public IP address from which Azure will allow your PostgreSQL connection.

---

# Step 3 — Configure PostgreSQL Firewall

In Azure Portal:

**PostgreSQL Flexible Server → Networking**

You should see the firewall configuration.

Choose:

**Add current client IP address**

Azure should automatically detect your current public IP.

You'll see something similar to:

```text
Firewall rule name: ClientIPAddress
Start IP address:   49.xx.xx.xx
End IP address:     49.xx.xx.xx
```

Click:

**Save**

### Important

We are deliberately allowing **only your current IP**, rather than opening PostgreSQL to the entire Internet.

PostgreSQL uses:

```text
TCP 5432
```

So the connection is:

```text
Your PC
   |
   | TCP 5432
   |
   v
Azure Firewall
   |
   | Your IP allowed?
   |
   +---- YES ----> PostgreSQL
   |
   +---- NO -----> Block
```

---

# Step 4 — Install PostgreSQL Client

You don't need to install the PostgreSQL database server on your laptop.

You only need the **PostgreSQL client**, particularly:

```text
psql
```

### Option A — If PostgreSQL is already installed

Open PowerShell:

```powershell
psql --version
```

If you see something like:

```text
psql (PostgreSQL) 17.x
```

you're ready.

### Option B — Install PostgreSQL

Download PostgreSQL for Windows from the official PostgreSQL site:

[PostgreSQL Downloads](https://www.postgresql.org/download/windows/?utm_source=chatgpt.com)

During installation, make sure the PostgreSQL command-line tools are installed.

Then open a **new PowerShell window** and run:

```powershell
psql --version
```

---

# Step 5 — Connect to Azure PostgreSQL

Now use the server name from Azure.

For example:

```powershell
psql "host=postgres-demo-xxxx.postgres.database.azure.com port=5432 dbname=postgres user=postgresadmin sslmode=require"
```

It will ask:

```text
Password for user postgresadmin:
```

Enter the password you created in **Lesson 1**.

> Nothing will appear on the screen while you type the password. That's normal.

---

# Step 6 — Successful Connection

If everything works, you'll see something similar to:

```text
psql (17.x)
SSL connection (protocol: TLSv1.3, ...)
Type "help" for help.

postgres=>
```

🎉 **You are now connected to PostgreSQL running inside Azure.**

Notice the prompt:

```text
postgres=>
```

This means you're inside the PostgreSQL command-line interface.

---

# Step 7 — Verify the Connection

Run:

```sql
SELECT version();
```

You should get PostgreSQL version information.

Then:

```sql
SELECT current_database();
```

Expected:

```text
 current_database
------------------
 postgres
```

Now check the current user:

```sql
SELECT current_user;
```

You should get:

```text
 postgresadmin
```

---

# Step 8 — Check the Azure Server

Run:

```sql
SELECT inet_server_addr(), inet_server_port();
```

You can also run:

```sql
SELECT now();
```

---

# Step 9 — Explore PostgreSQL Commands

These are **psql commands**, not SQL commands.

List databases:

```text
\l
```

List users/roles:

```text
\du
```

Show current connection:

```text
\conninfo
```

List tables:

```text
\dt
```

Show help:

```text
\?
```

Exit:

```text
\q
```

---

# Step 10 — Understand What We Just Did

We have now established this connection:

```text
                 Azure
┌──────────────────────────────────────┐
│                                      │
│   PostgreSQL Flexible Server         │
│                                      │
│   postgres-demo-xxxx                 │
│            │                         │
│            │ TCP 5432                │
│            │                         │
└────────────┼─────────────────────────┘
             │
             │ Internet
             │
             ↓
       Your Public IP
             │
             ↓
       Windows PC
          psql
```

The important pieces are:

| Component                        | Purpose                        |
| -------------------------------- | ------------------------------ |
| Azure PostgreSQL Flexible Server | Runs PostgreSQL                |
| Public endpoint                  | Allows network connectivity    |
| Firewall rule                    | Controls who can connect       |
| TCP 5432                         | PostgreSQL network port        |
| `psql`                           | Command-line PostgreSQL client |
| `postgresadmin`                  | PostgreSQL administrator       |
| SSL                              | Encrypts the connection        |

---

## 🧪 Your Exercise

Don't move to the next lesson yet.

Run these **five commands**:

```sql
SELECT version();
```

```sql
SELECT current_database();
```

```sql
SELECT current_user;
```

```text
\conninfo
```

```text
\l
```

Then tell me **what output you get** (you can paste it here).

After that, **Lesson 3** will be:

> **Create a database → create users/roles → create tables → INSERT → SELECT → UPDATE → DELETE**

That's where we'll start actually using PostgreSQL.
