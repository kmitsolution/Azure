# Azure Container Apps – Revision + Scaling

This is an important concept because **each revision can have its own scaling configuration**.

The easiest way to remember:

> **Revision = version of the application**
> **Scaling = number of replicas for that revision**

---

## 1. Simple Example

Suppose we have:

```text
Container App:
myapp
```

Two revisions:

```text
Revision 1 → myapp--nginx-v1
Revision 2 → myapp--httpd-v2
```

We can configure different scaling limits for each revision:

```text
Nginx Revision
    Min replicas = 2
    Max replicas = 5

HTTPD Revision
    Min replicas = 1
    Max replicas = 3
```

Architecture:

```text
                         myapp
                           │
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
       nginx-v1 revision           httpd-v2 revision
             │                           │
          2–5 replicas                1–3 replicas
             │                           │
       ┌─────┴─────┐                ┌────┴────┐
       ▼           ▼                ▼         ▼
      R1          R2               R1        R2
```

The important point is:

```text
nginx-v1 → its own scaling configuration

httpd-v2 → its own scaling configuration
```

---

# 2. Why Would We Want Different Scaling?

Imagine:

```text
Nginx v1 = Production
HTTPD v2 = Testing
```

Production receives a lot of traffic:

```text
Nginx:
Minimum = 3
Maximum = 10
```

Testing receives very little traffic:

```text
HTTPD:
Minimum = 1
Maximum = 2
```

So:

```text
Production
   ↓
nginx-v1
   ↓
3–10 replicas


Testing
   ↓
httpd-v2
   ↓
1–2 replicas
```

This allows each revision to scale independently.

---

# 3. Scaling Is Revision-Specific

Suppose:

```text
myapp--nginx-v1
```

has:

```text
Min = 2
Max = 5
```

Then you deploy:

```text
myapp--httpd-v2
```

and configure:

```text
Min = 1
Max = 3
```

You now have:

```text
Revision                 Min       Max
------------------------------------------------
nginx-v1                  2         5
httpd-v2                  1         3
```

These configurations belong to their respective revisions.

---

# 4. CLI Example

For our `myapp`, configure Nginx:

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --min-replicas 2 `
  --max-replicas 5 `
  --revision-suffix nginx-scale-v1
```

This creates/updates a revision with:

```text
Minimum replicas = 2
Maximum replicas = 5
```

For a new revision, you can configure a different scaling range.

For example:

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --image httpd:alpine `
  --min-replicas 1 `
  --max-replicas 3 `
  --revision-suffix httpd-scale-v2
```

Now conceptually:

```text
nginx-scale-v1
    Min = 2
    Max = 5

httpd-scale-v2
    Min = 1
    Max = 3
```

---

# 5. Very Important: Scaling Can Create a New Revision

This is an important point for students.

Some Container App configuration changes are **revision-scope changes**.

For example, changing:

```text
Image
CPU / Memory
Environment variables
Commands
Scale configuration
```

can result in a new revision.

So think:

```text
Change application/revision configuration
              ↓
        New revision
              ↓
    New scaling configuration
```

---

# 6. Revision + Traffic Splitting + Scaling

Now let's combine the concepts.

Suppose:

```text
Revision 1 → Nginx
Revision 2 → HTTPD
```

Traffic:

```text
Nginx = 80%
HTTPD = 20%
```

Scaling:

```text
Nginx:
Min = 2
Max = 10

HTTPD:
Min = 1
Max = 5
```

Architecture:

```text
                         Main URL
                            │
                 ┌──────────┴──────────┐
                80%                    20%
                 │                      │
                 ▼                      ▼
             Nginx v1                HTTPD v2
                 │                      │
              2–10 replicas           1–5 replicas
```

Therefore, **traffic percentage and replica count are separate concepts**.

---

# 7. 80/20 Does NOT Mean 80/20 Replicas

This is a very important teaching point.

Suppose:

```text
Traffic:
Nginx = 80%
HTTPD = 20%
```

It does **not** mean:

```text
Nginx = 80 replicas
HTTPD = 20 replicas
```

Instead:

```text
Traffic distribution
        ↓
80% → Nginx
20% → HTTPD
```

And scaling independently determines:

```text
Nginx → 2–10 replicas
HTTPD → 1–5 replicas
```

So:

```text
Traffic Splitting
       ≠
Replica Scaling
```

---

# 8. Example During Canary Deployment

This makes a great real-world example.

### Step 1 – Production

```text
Nginx v1
Traffic = 100%

Min = 3
Max = 10
```

```text
Nginx v1
 ├── Replica 1
 ├── Replica 2
 └── Replica 3
```

---

### Step 2 – Deploy New Version

```text
Nginx v1 → Production
HTTPD v2 → New version
```

Configure:

```text
Nginx:
Min = 3
Max = 10

HTTPD:
Min = 1
Max = 3
```

Then:

```text
Nginx = 80%
HTTPD = 20%
```

---

### Step 3 – Increase Traffic

If the new version performs well:

```text
Nginx = 50%
HTTPD = 50%
```

HTTPD can independently scale:

```text
1 → 2 → 3 replicas
```

according to its scaling configuration and workload.

---

### Step 4 – Full Rollout

Finally:

```text
Nginx = 0%
HTTPD = 100%
```

Now HTTPD is the production revision.

---

# 9. Activation vs Scaling

Don't confuse these concepts.

### Activation

```text
Revision
   ↓
Active / Inactive
```

Controls whether the revision is running.

### Scaling

```text
Active Revision
       ↓
Number of replicas
       ↓
1, 2, 3, 4...
```

Controls how many instances run.

### Traffic

```text
Main URL
    ↓
Traffic percentage
    ↓
Revision
```

Controls how requests are distributed.

---

# 10. Simple Diagram

```text
                       Container App
                            │
                 ┌──────────┴──────────┐
                 │                     │
              Revision 1            Revision 2
              nginx-v1              httpd-v2
                 │                     │
            Scaling config        Scaling config
              2–10                   1–5
                 │                     │
           ┌─────┴─────┐          ┌────┴────┐
           ▼           ▼          ▼         ▼
         Replica     Replica    Replica   Replica
            1           2          1         2
```

---

# 11. Key Concepts to Remember

| Concept               | Meaning                                      |
| --------------------- | -------------------------------------------- |
| **Revision**          | Version of the Container App                 |
| **Replica**           | Running instance of a revision               |
| **Min replicas**      | Minimum number of replicas                   |
| **Max replicas**      | Maximum number of replicas                   |
| **Traffic splitting** | Percentage of requests sent to each revision |
| **Activation**        | Start a revision                             |
| **Deactivation**      | Stop a revision                              |

### Memory Trick

```text
Revision
   ↓
Which version?

Scaling
   ↓
How many replicas?

Traffic splitting
   ↓
How much traffic?

Activation
   ↓
Should it be running?
```

**One-line explanation for your lesson:**

> **Each revision can have its own replica and scaling configuration, allowing different versions of the application to scale independently based on their workload.**
