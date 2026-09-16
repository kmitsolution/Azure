# Azure Container Apps – Blue-Green Deployment

**Blue-Green Deployment** is a deployment strategy where you keep the **current production revision (Blue)** and the **new revision (Green)** running at the same time.

You test Green first, and when you're satisfied, you switch production traffic from Blue to Green.

Azure Container Apps revisions and traffic splitting support this deployment pattern. ([Microsoft Learn][1])

---

## 1. Simple Definition

Think of it as:

```text
BLUE  = Current Production
GREEN = New Version
```

Example:

```text
Blue
  ↓
nginx-v1

Green
  ↓
httpd-v2
```

Initially:

```text
100% → Blue
  0% → Green
```

After successful testing:

```text
  0% → Blue
100% → Green
```

---

# 2. Our Simple Example

We'll use your existing Container App:

```text
Resource Group:
MYRG-India

Container App:
myapp
```

Two revisions:

```text
Blue
myapp--nginx-v1
nginx:alpine

Green
myapp--httpd-v2
httpd:alpine
```

Architecture:

```text
                     myapp
                       │
              ┌────────┴────────┐
              │                 │
           BLUE              GREEN
              │                 │
          nginx-v1           httpd-v2
```

---

# 3. Step 1 – Enable Multiple Revision Mode

Blue-Green deployment requires both revisions to be active simultaneously, so use **Multiple revision mode**. Microsoft documents multiple mode as the mode used when you need control over multiple active revisions and traffic allocation. ([Microsoft Learn][2])

```powershell
az containerapp revision set-mode `
  --name myapp `
  --resource-group MYRG-India `
  --mode multiple
```

---

# 4. Step 2 – Make Blue the Production Revision

Assume:

```text
Blue = myapp--nginx-v1
```

Set:

```text
Blue   = 100%
Green  = 0%
```

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=100 myapp--httpd-v2=0
```

Architecture:

```text
                     Users
                       │
                       ▼
                    myapp
                       │
                     100%
                       │
                       ▼
                 BLUE - nginx
                  v1 revision
```

Green exists, but production traffic isn't being sent to it.

---

# 5. Step 3 – Activate Green

Make sure the Green revision is active:

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2
```

Now:

```text
Blue  → Active
Green → Active
```

But traffic is still:

```text
Blue  = 100%
Green = 0%
```

This is important.

> **Active does not mean receiving traffic.**

An active revision can be running while receiving 0% of the main application traffic.

---

# 6. Step 4 – Test Green

Now you want to test:

```text
Green
  ↓
httpd-v2
```

You can use a **revision label** for direct access.

For example:

```text
staging → myapp--httpd-v2
```

Create it:

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --label staging
```

Then use the staging URL:

```text
https://myapp---staging.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

This lets you test Green without changing the production traffic. Revision labels provide a dedicated URL that routes directly to the labeled revision. ([Microsoft Learn][3])

---

# 7. Blue-Green Architecture During Testing

```text
                         Users
                           │
                           ▼
                     Main URL
                           │
                         100%
                           │
                           ▼
                    BLUE - Nginx
                    myapp--nginx-v1


                    Test Users
                         │
                         ▼
                  Staging URL
                         │
                         ▼
                   GREEN - HTTPD
                   myapp--httpd-v2
```

So:

```text
Production → Blue
Testing    → Green
```

This is the heart of Blue-Green deployment.

---

# 8. Step 5 – Switch Production to Green

Once Green has been tested successfully, change:

```text
Blue  = 100%
Green = 0%
```

to:

```text
Blue  = 0%
Green = 100%
```

Run:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=0 myapp--httpd-v2=100
```

Now:

```text
                         Users
                           │
                           ▼
                        myapp
                           │
                         100%
                           │
                           ▼
                    GREEN - HTTPD
                    myapp--httpd-v2
```

Traffic splitting weights must total 100%. Azure Container Apps supports assigning traffic percentages to active revisions. ([Microsoft Learn][4])

---

# 9. What Happened to Blue?

Blue still exists:

```text
myapp--nginx-v1
```

but:

```text
Traffic = 0%
```

You now have:

```text
Blue
  Active
  Traffic = 0%

Green
  Active
  Traffic = 100%
```

This is useful because Blue can serve as your rollback version.

---

# 10. Rollback

Suppose Green has a problem after production traffic is switched.

Current:

```text
Blue  = 0%
Green = 100%
```

Immediately switch back:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=100 myapp--httpd-v2=0
```

Now:

```text
Blue  = 100%
Green = 0%
```

Traffic returns to Nginx.

Architecture:

```text
             Production
                  │
                  ▼
             BLUE - Nginx
                  │
                100%
```

This is one of the main benefits of keeping the old revision available during a Blue-Green deployment. Azure specifically documents rollback as part of its Container Apps Blue-Green deployment workflow. ([Microsoft Learn][1])

---

# 11. Complete Blue-Green Flow

This is the diagram I recommend using in your course:

```text
                  BLUE
              nginx-v1
                  │
                100%
                  │
                  ▼
                USERS


        Deploy GREEN revision

                  │
                  ▼

                 GREEN
               httpd-v2
                  │
               Testing
                  │
                  ▼
             Staging URL


        GREEN testing successful

                  │
                  ▼

          Switch Production

       BLUE             GREEN
        0%               100%
         │                 │
         X                 ▼
                     Production
                        Users
```

---

# 12. Blue-Green vs Canary

These are often confused.

### Blue-Green

You normally keep two versions:

```text
Blue  → Old
Green → New
```

Then perform a full switch:

```text
100/0
   ↓
0/100
```

### Canary

You gradually increase traffic:

```text
100/0
  ↓
80/20
  ↓
50/50
  ↓
20/80
  ↓
0/100
```

Traffic splitting supports both gradual rollouts and Blue-Green patterns. ([Microsoft Learn][5])

---

# 13. Blue-Green vs Traffic Splitting

Traffic splitting is a **mechanism**.

Blue-Green is a **deployment strategy**.

For example:

```text
Traffic Splitting
      │
      ├── 100/0
      ├── 80/20
      ├── 50/50
      └── 0/100
```

You can use those traffic controls to implement different deployment strategies.

Blue-Green:

```text
Blue  → 100%
Green → 0%

        ↓

Blue  → 0%
Green → 100%
```

Canary:

```text
Blue  → 80%
Green → 20%

        ↓

Blue  → 50%
Green → 50%

        ↓

Blue  → 0%
Green → 100%
```

---

# 14. Blue-Green with Revision Labels

This is particularly easy to demonstrate.

Create:

```text
production → Blue
staging    → Green
```

Initially:

```text
production
    ↓
Nginx v1

staging
    ↓
HTTPD v2
```

Test Green using:

```text
staging URL
```

When ready, you can move the `production` label to the Green revision.

```text
Before:

production → Nginx
staging    → HTTPD


After:

production → HTTPD
staging    → HTTPD
```

The production label URL remains the same while the revision behind it changes. ([Microsoft Learn][3])

---

# 15. Important Difference: Label vs Traffic

For your students, explain it this way:

```text
Revision Label
      ↓
Direct access to a specific revision
```

Example:

```text
staging URL
     ↓
Green
```

Traffic splitting:

```text
Main URL
    ↓
    ├── 80% Blue
    └── 20% Green
```

Blue-Green deployment:

```text
Old Version = Blue
New Version = Green

Test Green
    ↓
Switch production
    ↓
Green becomes production
```

---

# 16. Complete CLI Demo

### 1. Multiple revision mode

```powershell
az containerapp revision set-mode `
  --name myapp `
  --resource-group MYRG-India `
  --mode multiple
```

### 2. Set Blue to 100%

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=100 myapp--httpd-v2=0
```

### 3. Activate Green

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2
```

### 4. Create staging label

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --label staging
```

### 5. Test Green

```text
https://myapp---staging.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

### 6. Switch production to Green

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=0 myapp--httpd-v2=100
```

### 7. Rollback if required

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=100 myapp--httpd-v2=0
```

---

# 17. Blue-Green Memory Trick

Remember:

```text
BLUE
 ↓
Current Production

GREEN
 ↓
New Version
```

Then:

```text
Test GREEN
     ↓
GREEN Ready?
     ↓
YES
     ↓
Switch Traffic
     ↓
GREEN = Production
```

If something goes wrong:

```text
GREEN
  ↓
Problem
  ↓
Switch back
  ↓
BLUE = Production
```

---

## 18. Final Summary

| Step                | Blue |           Green |
| ------------------- | ---: | --------------: |
| Initial production  | 100% |              0% |
| Deploy new revision | 100% |              0% |
| Test Green          | 100% | 0% via main URL |
| Promote Green       |   0% |            100% |
| Rollback            | 100% |              0% |

**One-line explanation for your lesson:**

> **Blue-Green deployment keeps the current and new revisions available side-by-side, tests the new Green revision, and then switches production traffic from Blue to Green; if a problem occurs, traffic can be switched back to Blue.** ([Microsoft Learn][1])

[1]: https://learn.microsoft.com/en-us/azure/container-apps/blue-green-deployment?utm_source=chatgpt.com "Blue-Green Deployment in Azure Container Apps | Microsoft Learn"
[2]: https://learn.microsoft.com/th-th/Azure/container-apps/revisions?utm_source=chatgpt.com "Update and deploy changes in Azure Container Apps | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/azure/container-apps/revisions-manage?utm_source=chatgpt.com "Manage revisions in Azure Container Apps | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/cli/azure/containerapp/ingress/traffic?view=azure-cli-latest&utm_source=chatgpt.com "az containerapp ingress traffic | Microsoft Learn"
[5]: https://learn.microsoft.com/fil-ph/%20azure/container-apps/traffic-splitting?utm_source=chatgpt.com "Traffic splitting in Azure Container Apps | Microsoft Learn"
