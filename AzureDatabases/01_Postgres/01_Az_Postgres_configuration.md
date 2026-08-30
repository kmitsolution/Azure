# Lesson 1 — Create Azure Database for PostgreSQL

Today we'll **only create the PostgreSQL Flexible Server**. Don't configure the application or RBAC yet.

![Image](https://images.openai.com/static-rsc-4/zzXbGH73BEmUTNJiKHl6xepBXpavm7xCdAvitfLSxleBE4AYU2ayqRK6JfJ-Y8ZyMDB0rE51ZX-47V8Eey631O0h6mCvKqsuV0obtuxc9J9WHMZ5rHaKb-gQQZ6xW2zJ3FGWgBFcUhrVF2o2XcVybEIIq-h9pb3VbiNniGehAG9iy2hkZ9Hz2S6up4x0OBje?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/mZxJMTvJ1ohvjTP3gqndHe_14gQrn55SQnvU8OzbPj7JFVCF8uVkmoLoy8wDBcuQkJTQBMKB_oNYDAuUw28HGofApUAEQsn24d-GfN2cVXaw4sTm2s6HA1acEhcUxsOGEu6IIpbtb79YoyXVPJV2vugll6SNBqNYGO4BiDOrsNLrRlAsy0nhjKIL8uu6ljyx?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/K1iNcMzTIgFaRyxkcBAxVR0-o4aJTWNWGHFA9kHqanNyqXgueb1D0CFARIEIOdU7LROG6S4mGV5tvhYpJA6lrkkxOR-EZ3iMYh6m8_dMRM4vlFBpiCxn5s7a0d3ij3Uzmsy7WyAUkh5rdDDQLOiRSKdmZWCqiHSgnT84fecGceAsFRuCbs5mj1IQk_utOekj?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ryP5nOQ5BKzgk8SDowjJJqvtRDUI78T3t1pDCGMmPcOagxwZOI5ZCa458-x3uUXnHPJ3MbsFtBfwc9fqOA8THNRzggNbTMdvFDb0fdlRjsaaKod9GkcPooXNVi52FIn7IY9vJHvhcT5RGHFLXotz3kfE4OMHsHdlsMuuOTOp_-6HJcSkYjbxVgobgJ4nIF4m?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/-hTQ-zW98kcgBqwaUBQVvv3vM1vaZCTpnB3Iy3Mled4uAN1miRL7DOG1s6YFdT4IqNgQRgQIEx5lBzSbUcmI2TFvTkxht1TIUaW2l1zk-j-fm-7BvAfHzfwecnk16ZZdXTdedDsW5vh0rpcIKzqkzcWtmVFkjEiXBuU9NLaeCGxcW4uOSob0RSB_bBF7JcDC?purpose=fullsize)

## Step 1 — Open Azure Portal

Go to:

[Azure Portal](https://portal.azure.com/?utm_source=chatgpt.com)

Sign in with your Azure account.

---

## Step 2 — Create PostgreSQL Flexible Server

In the Azure Portal search bar, search:

```text
Azure Database for PostgreSQL servers
```

Select **Azure Database for PostgreSQL servers**.

Click:

**+ Create**

Choose:

**Azure Database for PostgreSQL Flexible Server**

---

## Step 3 — Basics

Configure the **Basics** tab like this:

| Setting            | Value                         |
| ------------------ | ----------------------------- |
| Subscription       | Your Azure subscription       |
| Resource group     | `rg-postgres-demo`            |
| Server name        | `postgres-demo-<unique-name>` |
| Region             | Choose a nearby region        |
| PostgreSQL version | Default/latest supported      |
| Workload type      | Development                   |
| High availability  | Disabled for learning         |

### Server name

The name must be globally unique.

For example:

```text
postgres-demo-2026
```

If that name is unavailable, use something like:

```text
postgres-demo-rs-2026
```

---

## Step 4 — Authentication

For **Authentication method**, select:

```text
PostgreSQL authentication only
```

Create an administrator:

```text
Administrator username:
postgresadmin
```

Set a strong password.

For example, **do not actually use this example password**:

```text
MyPostgres@12345
```

Save your password somewhere safe because we'll need it in the next lesson.

---

## Step 5 — Compute + Storage

For learning, select the smallest/lowest-cost option available.

Look for:

```text
Compute tier: Burstable
```

Then choose a small B-series configuration if available.

You don't need a large production server for this course.

For storage, leave the default unless Azure shows a significantly higher-cost configuration.

---

## Step 6 — Networking

For our **first hands-on lesson**, choose:

```text
Connectivity method:
Public access
```

Then enable:

```text
Allow public access from any Azure service within Azure to this server
```

For your own computer, we'll configure the firewall properly in the next step.

### Important

Do **not** leave the PostgreSQL server permanently open to:

```text
0.0.0.0 - 255.255.255.255
```

We'll learn how to secure this later.

---

## Step 7 — Review + Create

Click:

**Review + create**

Azure will validate the configuration.

If validation succeeds:

**Create**

Deployment may take a few minutes.

---

# Step 8 — Open the PostgreSQL Server

After deployment finishes, click:

**Go to resource**

You should see something similar to:

```text
Overview

Server name
postgres-demo-xxxx.postgres.database.azure.com

Server status
Ready

Location
...
```

### ⚠️ Save these two values

We will need them in Lesson 2:

```text
Server:
postgres-demo-xxxx.postgres.database.azure.com

Username:
postgresadmin
```

And keep your password secure.

---

# Step 9 — Verify the Server

On the PostgreSQL server's **Overview** page, confirm:

```text
Status: Ready
```

Then go to:

**Settings → Server parameters**

Don't change anything yet.

Just understand that this is where PostgreSQL configuration parameters can be managed.

---

## What you have created

Your architecture currently looks like:

```text
Your Computer
      |
      | Internet
      |
      v
+-----------------------------+
|        Azure                |
|                             |
|  Resource Group             |
|  rg-postgres-demo           |
|          |                  |
|          v                  |
|  PostgreSQL Flexible Server |
|                             |
|  postgres-demo-xxxx         |
|          |                  |
|          v                  |
|      PostgreSQL             |
+-----------------------------+
```

### Lesson 1 goal

You should now have:

```text
Azure Subscription
       ↓
Resource Group
       ↓
PostgreSQL Flexible Server
       ↓
PostgreSQL Database Service
```

**Stop here.**

Once the server shows **Ready**, tell me **"done"** and we'll do **Lesson 2: Configure networking/firewall and connect to PostgreSQL from your Windows machine using `psql`**.
