# Azure Container Apps – Revision Activation / Deactivation

**Revision Activation / Deactivation** allows you to **start or stop a specific revision** of an Azure Container App.

The easiest way to remember it:

```text
Activate   → Start the revision
Deactivate → Stop the revision
```

This is different from **traffic splitting**.

```text
Traffic Splitting
    ↓
How much traffic does a revision receive?

Activation / Deactivation
    ↓
Should the revision be running or stopped?
```

---

## 1. Simple Example

Assume our Container App is:

```text
Container App:
myapp
```

with two revisions:

```text
myapp--nginx-v1
myapp--httpd-v2
```

Initially:

```text
Nginx revision → Active
HTTPD revision → Active
```

We can deactivate HTTPD:

```text
Nginx revision → Active
HTTPD revision → Inactive
```

The HTTPD revision is no longer running and doesn't serve traffic.

---

# 2. Check Revision Status

Use:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You may see information similar to:

```text
Name                 Active
-------------------  ------
myapp--nginx-v1      True
myapp--httpd-v2      True
```

The exact columns can vary depending on Azure CLI version.

---

# 3. Deactivate a Revision

Suppose we want to stop:

```text
myapp--httpd-v2
```

Run:

```powershell
az containerapp revision deactivate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2
```

Now conceptually:

```text
myapp--nginx-v1
       ↓
     Active

myapp--httpd-v2
       ↓
    Inactive
```

---

# 4. Activate the Revision Again

When you want to start the revision again:

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2
```

Now:

```text
myapp--nginx-v1
       ↓
     Active

myapp--httpd-v2
       ↓
     Active
```

---

# 5. Complete Demo

Let's perform a simple demonstration.

### Step 1 – List revisions

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

Assume:

```text
myapp--nginx-v1
myapp--httpd-v2
```

---

### Step 2 – Deactivate HTTPD

```powershell
az containerapp revision deactivate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2
```

Now:

```text
Nginx → Active
HTTPD → Inactive
```

---

### Step 3 – Verify

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

---

### Step 4 – Activate HTTPD

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2
```

Now:

```text
Nginx → Active
HTTPD → Active
```

---

# 6. Activation Is Not Traffic Splitting

This is an important distinction.

Suppose:

```text
Nginx → Active
HTTPD → Active
```

but traffic is:

```text
Nginx = 100%
HTTPD = 0%
```

HTTPD is **running**, but it isn't receiving traffic from the main application endpoint.

Now suppose:

```text
Nginx → Active
HTTPD → Inactive
```

You cannot send traffic to HTTPD while it is inactive.

Think of it as:

```text
              Revision
                 │
        ┌────────┴────────┐
        │                 │
     Active            Inactive
        │                 │
        ▼                 X
   Can run/serve       Stopped
```

---

# 7. Activation + Traffic Splitting

These two features can work together.

For example:

```text
Nginx → Active
HTTPD → Active
```

Then configure:

```text
Nginx = 80%
HTTPD = 20%
```

Architecture:

```text
                  Main URL
                     │
              ┌──────┴──────┐
             80%            20%
              │              │
              ▼              ▼
           Nginx            HTTPD
            v1                v2
             ▲                 ▲
             │                 │
           Active            Active
```

If HTTPD is deactivated:

```text
Nginx → Active
HTTPD → Inactive
```

then HTTPD cannot serve requests.

---

# 8. Activation + Revision Labels

This is also useful with your previous lesson.

Suppose:

```text
production → Nginx
staging    → HTTPD
```

and both revisions are active:

```text
Nginx → Active
HTTPD → Active
```

Testers can access:

```text
staging URL
       ↓
HTTPD
```

If you deactivate HTTPD:

```text
HTTPD → Inactive
```

the staging revision is stopped.

You can later activate it again:

```text
HTTPD → Active
```

---

# 9. Real-World Use Case

Imagine:

```text
Revision 1 → Production
Revision 2 → New version
```

You don't need Revision 2 running all the time.

You could:

```text
Revision 1 → Active
Revision 2 → Inactive
```

When you're ready for testing:

```text
Activate Revision 2
```

Test it.

Then decide whether to:

```text
Increase traffic
```

or:

```text
Deactivate Revision 2
```

This can help keep old/non-production revisions from continuously consuming resources.

---

# 10. Important Difference: Deactivate vs Delete

These are **not the same**.

### Deactivate

```text
Revision
   ↓
Inactive
```

The revision still exists.

You can activate it again.

### Delete

```text
Revision
   ↓
Deleted
```

The revision is removed.

So:

```text
Deactivate = Stop it
Delete     = Remove it
```

For example:

```text
myapp--httpd-v2
        │
        ├── deactivate → still exists
        │
        └── delete     → removed
```

---

# 11. Useful Commands

### List all revisions

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

### Activate

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision <REVISION_NAME>
```

### Deactivate

```powershell
az containerapp revision deactivate `
  --name myapp `
  --resource-group MYRG-India `
  --revision <REVISION_NAME>
```

---

## Simple Memory Trick

```text
ACTIVATE
   ↓
Start / Run the revision

DEACTIVATE
   ↓
Stop the revision

TRAFFIC SPLIT
   ↓
Control percentage of traffic
```

### One-line explanation for your AZ-104/ACA lesson

> **Revision Activation/Deactivation controls whether a specific revision is running, while traffic splitting controls how much traffic an active revision receives.**
