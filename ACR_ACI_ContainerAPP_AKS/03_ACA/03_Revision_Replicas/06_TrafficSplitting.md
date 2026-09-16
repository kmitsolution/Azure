**Azure Container Apps revision management**, **traffic splitting** controls what percentage of requests go to each active revision.

### Simple Example

Assume:

```text
Container App: myapp

Revision 1 → myapp--nginx-v1
Revision 2 → myapp--httpd-v2
```

Your main application URL is:

```text
https://myapp.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

## 1. 100/0 — All Traffic to Nginx

```text
                 Main URL
                    │
                  100%
                    │
                    ▼
             myapp--nginx-v1
                    │
                    ▼
                  Nginx
```

Configuration:

```text
Nginx = 100%
HTTPD = 0%
```

CLI:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=100 myapp--httpd-v2=0
```

**Use case:** New HTTPD revision exists, but you haven't exposed it to users yet.

---

## 2. 80/20 — Canary Deployment

```text
                 Main URL
                    │
             ┌──────┴──────┐
            80%            20%
             │              │
             ▼              ▼
          Nginx           HTTPD
           v1               v2
```

Configuration:

```text
Nginx = 80%
HTTPD = 20%
```

CLI:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=80 myapp--httpd-v2=20
```

### What does 80/20 mean?

It does **not** mean:

```text
First 80 users → Nginx
Next 20 users → HTTPD
```

Instead, requests are distributed according to the configured weights over time.

For example, across a sufficiently large number of requests, you would expect approximately:

```text
80% → Nginx
20% → HTTPD
```

**Use case:** Canary deployment.

---

# 3. 50/50 — Equal Traffic

```text
                 Main URL
                    │
             ┌──────┴──────┐
            50%            50%
             │              │
             ▼              ▼
          Nginx           HTTPD
           v1               v2
```

CLI:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=50 myapp--httpd-v2=50
```

Configuration:

```text
Nginx = 50%
HTTPD = 50%
```

**Use case:** A/B testing or running two versions with equal traffic.

---

# 4. Common Progression

A very good real-world demo is:

```text
Step 1
100/0

Nginx = 100%
HTTPD = 0%
```

Then deploy HTTPD:

```text
Step 2
80/20

Nginx = 80%
HTTPD = 20%
```

If everything looks good:

```text
Step 3
50/50

Nginx = 50%
HTTPD = 50%
```

Then finally:

```text
Step 4
0/100

Nginx = 0%
HTTPD = 100%
```

Diagram:

```text
100/0
  │
  ▼
80/20
  │
  ▼
50/50
  │
  ▼
0/100
```

This is a simple way to demonstrate a **gradual rollout**.

---

# 5. Rollback

Suppose you reach:

```text
HTTPD = 100%
Nginx = 0%
```

but you want to immediately return to Nginx:

```text
Nginx = 100%
HTTPD = 0%
```

Run:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=100 myapp--httpd-v2=0
```

So traffic splitting makes rollback very straightforward.

---

## Important: Main URL vs Label URL

This is the key concept from your previous **Direct Revision Access** lesson:

```text
Traffic Splitting
─────────────────
Main URL
   │
   ├── 80% → Revision 1
   └── 20% → Revision 2
```

Whereas:

```text
Revision Label
───────────────
staging URL
   │
   └──→ Specific Revision
```

So:

> **Traffic splitting = distribute main URL traffic between revisions.**

> **Revision label = directly access a specific revision through a dedicated URL.**

### Quick teaching table

| Traffic   | Meaning          | Typical Use  |
| --------- | ---------------- | ------------ |
| **100/0** | 100% old, 0% new | Production   |
| **80/20** | 80% old, 20% new | Canary       |
| **50/50** | Equal traffic    | A/B testing  |
| **0/100** | 0% old, 100% new | Full rollout |
