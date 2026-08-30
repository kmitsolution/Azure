1. **Create Azure PostgreSQL Flexible Server**
2. Connect using `psql`
3. Create database, users, tables
4. Perform CRUD operations
5. Understand PostgreSQL networking
6. Configure firewall/private access
7. Connect from an application
8. Backups and restore
9. Monitoring
10. Azure RBAC and Microsoft Entra authentication
11. Connect PostgreSQL with Azure VM / App Service / AKS
12. Production architecture

### Lesson 1 — Create PostgreSQL in Azure

In the Azure Portal:

**Azure Portal → Create a resource → search `Azure Database for PostgreSQL` → Flexible Server**

Use something like:

| Setting            | Example                        |
| ------------------ | ------------------------------ |
| Subscription       | Your Azure subscription        |
| Resource group     | `rg-postgres-demo`             |
| Server name        | `postgres-demo-<unique>`       |
| Region             | Your preferred region          |
| PostgreSQL version | Latest supported version       |
| Workload type      | Development                    |
| Compute            | Burstable / smallest available |
| Authentication     | PostgreSQL authentication      |
| Admin username     | `postgresadmin`                |
| Password           | Choose your password           |

For learning, keep the server configuration **small** so you don't unnecessarily spend money.

After deployment, open:

**PostgreSQL server → Overview**

You will see a server hostname similar to:

```text
postgres-demo.postgres.database.azure.com
```

### Lesson 2 — Connect from your computer

Install PostgreSQL client tools on your machine.

Then connect:

```bash
psql "host=postgres-demo.postgres.database.azure.com port=5432 dbname=postgres user=postgresadmin sslmode=require"
```

It will ask for the password you created.

If successful, you should see:

```text
postgres=>
```

Now test:

```sql
SELECT version();
```

and:

```sql
SELECT current_database();
```

### Lesson 3 — Create your first database

Inside `psql`:

```sql
CREATE DATABASE companydb;
```

List databases:

```sql
\l
```

Connect to it:

```sql
\c companydb
```

Create a table:

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(100),
    salary NUMERIC(10,2)
);
```

Insert data:

```sql
INSERT INTO employees (name, department, salary)
VALUES
('Ravi', 'DevOps', 75000),
('Anil', 'Development', 85000),
('Priya', 'Testing', 70000);
```

Query:

```sql
SELECT * FROM employees;
```

---

**I suggest we don't do everything at once.** We can build this as a hands-on course:

> **Azure PostgreSQL → Database → Networking → Application → Security → Entra ID → RBAC → Production**

If you already have an Azure subscription, **start with Lesson 1 and tell me when the PostgreSQL Flexible Server is created**. Then I'll give you only the next step.
