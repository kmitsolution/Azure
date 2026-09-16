# Azure Container Apps – Replicas in a Revision

A **revision** is a version of your Container App, while a **replica** is a running instance of that revision.

The easiest way to remember:

```text
Revision = Version
Replica  = Running copy
```

---

## 1. Simple Example

Suppose your Container App is:

```text
myapp
```

You have one revision:

```text
myapp--nginx-v1
```

Now configure:

```text
Minimum replicas = 2
Maximum replicas = 5
```

Azure Container Apps may run:

```text
Revision: myapp--nginx-v1

        ┌──────────────┐
        │   Revision   │
        │   nginx-v1   │
        └──────┬───────┘
               │
       ┌───────┼────────┐
       │       │        │
       ▼       ▼        ▼
    Replica 1 Replica 2 Replica 3
```

Each replica is a separate running instance of the same revision.

---

# 2. Revision vs Replica

This is very important.

### Revision

A revision represents a **specific version/configuration** of your Container App.

Example:

```text
myapp--nginx-v1
```

Then you deploy a new version:

```text
myapp--httpd-v2
```

Now you have:

```text
myapp
 │
 ├── Revision 1 → nginx-v1
 │
 └── Revision 2 → httpd-v2
```

### Replicas

Each revision can have multiple running instances.

For example:

```text
Revision 1 → nginx-v1

   ├── Replica 1
   ├── Replica 2
   └── Replica 3
```

So:

```text
1 Revision
   ↓
3 Replicas
```

---

# 3. Very Simple Real-World Example

Imagine your website receives a lot of traffic.

One container:

```text
Nginx
```

may not be enough.

So Azure runs three replicas:

```text
                 Users
                   │
                   ▼
             Container App
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
    Replica 1  Replica 2  Replica 3
       Nginx       Nginx      Nginx
```

All three replicas belong to the **same revision**.

```text
Revision
myapp--nginx-v1

        ↓
 ┌──────┼──────┐
 ▼      ▼      ▼
R1      R2      R3
```

---

# 4. What Happens When You Create a New Revision?

Suppose initially:

```text
Revision 1
myapp--nginx-v1

Replicas:
R1
R2
```

So:

```text
nginx-v1
   ├── Replica 1
   └── Replica 2
```

Now you deploy HTTPD:

```text
Revision 2
myapp--httpd-v2
```

You could have:

```text
nginx-v1
   ├── Replica 1
   └── Replica 2

httpd-v2
   ├── Replica 1
   └── Replica 2
```

Notice that the replicas belong to **different revisions**.

---

# 5. Revision + Replica + Traffic Splitting

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

And each revision has 2 replicas:

```text
                       Main URL
                          │
                 ┌────────┴────────┐
                80%               20%
                 │                 │
                 ▼                 ▼
          Nginx Revision     HTTPD Revision
                 │                 │
             ┌───┴───┐         ┌───┴───┐
             ▼       ▼         ▼       ▼
            R1      R2        R1       R2
```

So we have:

```text
2 Revisions
4 total replicas
```

---

# 6. Scaling a Revision

Suppose HTTPD starts receiving more traffic.

Azure Container Apps can increase the number of replicas according to the configured scaling rules.

Initially:

```text
HTTPD Revision

Replica 1
Replica 2
```

Later:

```text
HTTPD Revision

Replica 1
Replica 2
Replica 3
Replica 4
Replica 5
```

The revision hasn't changed.

Only the number of replicas changed.

This distinction is very important:

```text
Scaling
   ↓
Changes number of replicas

Deployment
   ↓
Creates a new revision
```

---

# 7. Minimum and Maximum Replicas

You can configure:

```text
Min replicas = 2
Max replicas = 5
```

Meaning conceptually:

```text
Minimum:
2 replicas

Maximum:
5 replicas
```

The actual number running can vary between those limits based on the configured scaling rules.

For example:

```text
Low traffic
    ↓
2 replicas

More traffic
    ↓
3 replicas

High traffic
    ↓
5 replicas
```

---

# 8. Simple CLI Example

You can configure replica limits using:

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --min-replicas 2 `
  --max-replicas 5
```

Now your application is configured for:

```text
Minimum = 2
Maximum = 5
```

---

# 9. Important: Replica Is Not Another Revision

Students often confuse this.

### Wrong idea

```text
Replica 1 = Revision 1
Replica 2 = Revision 2
```

❌ Not correct.

### Correct

```text
Revision 1
   │
   ├── Replica 1
   ├── Replica 2
   └── Replica 3
```

And:

```text
Revision 2
   │
   ├── Replica 1
   └── Replica 2
```

So:

> **Multiple replicas can belong to the same revision.**

---

# 10. Revision vs Replica – Easy Table

| Concept           | Meaning                               | Example           |
| ----------------- | ------------------------------------- | ----------------- |
| Container App     | Application                           | `myapp`           |
| Revision          | Version of application                | `myapp--nginx-v1` |
| Replica           | Running instance                      | Replica 1         |
| Scaling           | Changes number of replicas            | 2 → 5             |
| Traffic splitting | Distributes traffic between revisions | 80/20             |

---

# 11. Easy Memory Trick

Remember this:

```text
Container App
      ↓
   Revision
      ↓
   Replicas
```

Example:

```text
myapp
  │
  └── nginx-v1
         │
         ├── Replica 1
         ├── Replica 2
         └── Replica 3
```

### One-line explanation for your lesson:

> **A revision represents a version of the Container App, while replicas are the running instances of that revision.**
