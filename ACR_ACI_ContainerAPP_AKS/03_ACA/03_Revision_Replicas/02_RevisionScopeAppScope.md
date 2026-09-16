# Azure Container Apps – Revision Scope vs Application Scope

This is a very important concept because it explains **why some changes create a new revision and some changes do not**.

We'll use our simple:

```text
Container App: myapp
Image: nginx:alpine
```

---

## 1. Simple Definition

### Revision Scope

A **revision-scope change** creates a **new revision**.

Think:

> "I am changing the version of my application."

Examples:

```text
Container image
CPU / Memory
Environment variables
Container command
Container arguments
Scale configuration
Volume mounts
```

Example:

```text
Revision 1
nginx:alpine
      ↓
Change image
      ↓
Revision 2
httpd:alpine
```

---

### Application Scope

An **application-scope change** changes the Container App configuration but **does not create a new revision**.

Think:

> "I am changing how the existing application is exposed or managed."

Examples include:

```text
Ingress configuration
Traffic distribution
Revision mode
Revision labels
```

The exact set of revision- and application-scope properties is documented by Microsoft and can vary as ACA evolves.

---

# 2. Very Simple Example

Start with:

```text
myapp
   │
   └── Revision 1
          │
          ▼
      nginx:alpine
```

Now change the image:

```text
nginx:alpine
       ↓
httpd:alpine
```

This changes the application version.

Therefore Azure creates:

```text
myapp
   │
   ├── Revision 1 → nginx
   │
   └── Revision 2 → httpd
```

**Image change = Revision Scope**

---

# 3. Application Scope Example

Suppose we already have:

```text
Revision 1 → Nginx
Revision 2 → HTTPD
```

Now we change traffic:

```text
Before:

Nginx  → 100%
HTTPD  →   0%
```

to:

```text
After:

Nginx  → 80%
HTTPD  → 20%
```

We are **not changing either application version**.

We are only changing:

> "How should traffic be distributed?"

Therefore, this is an **application-scope change**.

No new revision is required just to change the traffic weights.

---

# 4. Easy Diagram

```text
                 Container App
                     myapp
                       │
              ┌────────┴────────┐
              │                 │
          Revision 1        Revision 2
           Nginx              HTTPD
              │                 │
              └────────┬────────┘
                       │
                 Traffic Rule
                       │
                  80% / 20%
```

The revisions stay the same.

Only the traffic configuration changes.

---

# 5. Revision Scope Example

Let's start:

```text
Revision 1
Image = nginx:alpine
```

Run:

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --image httpd:alpine
```

Azure creates a new revision:

```text
Revision 1 → nginx:alpine
Revision 2 → httpd:alpine
```

So:

```text
Image change
     ↓
Revision-scope change
     ↓
New revision
```

---

# 6. Application Scope Example – Traffic

Now assume:

```text
Revision 1 → Nginx
Revision 2 → HTTPD
```

Set:

```text
Nginx → 80%
HTTPD → 20%
```

CLI:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight <NGINX_REVISION>=80 <HTTPD_REVISION>=20
```

This changes traffic configuration.

It does **not** create another application version.

So we still have:

```text
Revision 1 → Nginx
Revision 2 → HTTPD
```

not:

```text
Revision 3
```

---

# 7. Another Easy Example – Revision Mode

Suppose:

```text
myapp

Revision 1 → Nginx
Revision 2 → HTTPD
```

We change:

```text
Single
```

to:

```text
Multiple
```

This changes the **revision management behavior** of the Container App.

It does not mean:

```text
Revision 3
```

is created.

Instead, the application changes how it manages active revisions.

---

# 8. Compare Them

| Change                                 | Scope       | New Revision? |
| -------------------------------------- | ----------- | ------------- |
| Change `nginx:alpine` → `httpd:alpine` | Revision    | ✅ Yes         |
| Change CPU                             | Revision    | ✅ Yes         |
| Change memory                          | Revision    | ✅ Yes         |
| Change environment variable            | Revision    | ✅ Yes         |
| Change container command               | Revision    | ✅ Yes         |
| Change volume mount                    | Revision    | ✅ Yes         |
| Change traffic 100% → 80/20            | Application | ❌ No          |
| Change revision mode                   | Application | ❌ No          |
| Change revision label                  | Application | ❌ No          |

---

# 9. Best Memory Trick

Remember:

### Revision Scope

**"What is inside my container?"**

```text
Image
CPU
Memory
Environment variables
Command
Arguments
Volumes
Scaling
```

These describe the **version of the workload**.

```text
Change it
   ↓
New Revision
```

### Application Scope

**"How is my application managed/exposed?"**

```text
Traffic
Revision mode
Labels
Ingress-related configuration
```

These generally affect the **Container App as a whole** rather than creating a new workload version.

---

# 10. Real-Life Deployment Example

Imagine:

```text
Revision 1
myapp:v1
```

You deploy:

```text
myapp:v2
```

Because the image changed:

```text
Revision 1 → v1
Revision 2 → v2
```

Now you want to test v2 with only 10% traffic:

```text
v1 → 90%
v2 → 10%
```

Changing the traffic percentage doesn't create:

```text
Revision 3
```

Instead:

```text
Revision 1 → v1 → 90%
Revision 2 → v2 → 10%
```

This is the core reason **revision scope and application scope** matter.

---

# 11. One-Line Summary

```text
Revision Scope
      ↓
Changes the application version
      ↓
New Revision
```

while:

```text
Application Scope
      ↓
Changes how the Container App is managed
      ↓
No new Revision
```

### For your Nginx/HTTPD lab

The easiest demonstration is:

```text
nginx:alpine
     │
     │ Change image
     ▼
httpd:alpine
     │
     ▼
NEW REVISION
```

Then:

```text
Nginx → 80%
HTTPD → 20%
     │
     ▼
Traffic configuration changes
     │
     ▼
NO NEW REVISION
```

That gives students a very clear distinction between the two concepts.
