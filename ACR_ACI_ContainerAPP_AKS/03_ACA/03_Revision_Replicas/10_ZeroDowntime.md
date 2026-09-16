# Azure Container Apps – Zero-Downtime Deployment

**Zero-downtime deployment** means deploying a new version of your Container App **without making the application unavailable to users**.

Azure Container Apps uses **revisions** to support this deployment model. In **single revision mode**, Azure keeps the existing revision serving traffic until the new revision is ready. ([Microsoft Learn][1])

---

## 1. Simple Example

Suppose your current production application is:

```text
myapp
  │
  ▼
Revision 1
nginx-v1
```

Users are accessing:

```text
https://myapp.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

Now you want to deploy:

```text
Revision 2
nginx-v2
```

We don't want this:

```text
Users
  │
  X
Application unavailable
  │
  ▼
Deploy v2
```

Instead, Azure does approximately:

```text
              Users
                │
                ▼
          Revision 1
           nginx-v1
                │
        Users continue
        receiving traffic
                │
                │
         Deploy Revision 2
                │
                ▼
          Revision 2
           nginx-v2
                │
        Start replicas
                │
        Health checks
                │
                ▼
          Ready ✓
                │
                ▼
        Traffic moves
                │
                ▼
          Revision 2
```

The old revision continues receiving traffic until the new revision is ready. ([Microsoft Learn][2])

---

# 2. Single Revision Mode

For a simple zero-downtime deployment demonstration, use:

```text
Single revision mode
```

Check the current mode:

```powershell
az containerapp show `
  --name myapp `
  --resource-group MYRG-India `
  --query properties.configuration.activeRevisionsMode `
  --output tsv
```

Set Single mode:

```powershell
az containerapp revision set-mode `
  --name myapp `
  --resource-group MYRG-India `
  --mode single
```

Azure's default revision mode is single. ([Microsoft Learn][3])

---

# 3. Deploy a New Version

Suppose the current image is:

```text
nginx:alpine
```

Now update the image:

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --image nginx:1.27-alpine
```

Because the image is a revision-scope change, Azure creates a new revision. ([Microsoft Learn][3])

You might now have:

```text
myapp--nginx-v1
myapp--nginx-v2
```

---

# 4. What Happens During Deployment?

Azure doesn't immediately stop the old revision.

Conceptually:

```text
Before deployment

Users
  │
  ▼
Revision 1
nginx-v1
```

During deployment:

```text
                    Users
                      │
                      ▼
                 Revision 1
                  nginx-v1
                      │
                      │ 100%
                      │
                      ▼

                 Revision 2
                  nginx-v2
                 Starting...
```

The old revision continues serving traffic.

The new revision starts its replicas and goes through provisioning and health checks.

---

# 5. When Is the New Revision Ready?

Azure considers the new revision ready when:

1. The revision provisions successfully.
2. Its replicas scale up appropriately.
3. Its replicas pass startup and readiness probes.

Only then does traffic move to the new revision in single revision mode. ([Microsoft Learn][2])

Think:

```text
New Revision
     │
     ▼
Provision
     │
     ▼
Start Replicas
     │
     ▼
Startup Probe
     │
     ▼
Readiness Probe
     │
     ▼
READY ✓
     │
     ▼
Traffic Switch
```

---

# 6. Health Probes Are Important

Suppose your new application starts slowly.

Azure needs to know:

> "Is this replica actually ready to receive traffic?"

That's where the **readiness probe** is important.

```text
New Replica
     │
     ▼
Readiness Probe
     │
     ├── FAIL → Don't send traffic
     │
     └── PASS → Ready for traffic
```

Microsoft recommends configuring appropriate startup, readiness, and liveness probes for reliable deployments. ([Microsoft Learn][4])

---

# 7. Simple Nginx Example

Let's use your existing example.

### Revision 1

```text
myapp--nginx-v1
```

Image:

```text
nginx:alpine
```

Suppose:

```text
Replicas = 2
```

```text
Revision 1
   │
   ├── Replica 1
   └── Replica 2
```

Now deploy Revision 2:

```text
myapp--nginx-v2
```

Azure starts the new revision.

```text
Revision 1
   ├── Replica 1
   └── Replica 2

Revision 2
   ├── Replica 1
   └── Replica 2
```

Once Revision 2 is ready:

```text
                    Users
                      │
                      ▼
                 Revision 2
                 nginx-v2
                      │
                ┌─────┴─────┐
                ▼           ▼
             Replica 1   Replica 2
```

The old revision can then be deactivated.

---

# 8. Why Replicas Matter

This connects directly with your previous lesson.

Suppose:

```text
Old Revision
Min replicas = 2
```

When deploying the new revision, Azure scales it appropriately before switching traffic, subject to the new revision's replica limits. ([Microsoft Learn][2])

So the deployment isn't simply:

```text
Stop old container
       ↓
Start new container
```

Instead, conceptually:

```text
Old revision running
        │
        ▼
Start new revision
        │
        ▼
New replicas become ready
        │
        ▼
Switch traffic
        │
        ▼
Old revision shuts down
```

This is the key idea behind zero-downtime deployment.

---

# 9. Zero-Downtime vs Traffic Splitting

These are related but different.

### Zero-Downtime Deployment

Goal:

```text
Keep application available
while deploying new version
```

### Traffic Splitting

Goal:

```text
Control how much traffic
each revision receives
```

For example:

```text
80% → v1
20% → v2
```

Traffic splitting is especially useful in **multiple revision mode** for controlled rollouts. ([Microsoft Learn][5])

---

# 10. Single Mode vs Multiple Mode

### Single Revision Mode

Azure manages the transition:

```text
v1
 │
 │ 100% traffic
 ▼
Deploy v2
 │
 ▼
v2 becomes ready
 │
 ▼
Traffic moves to v2
 │
 ▼
v1 deactivated
```

This provides the simple zero-downtime deployment experience. ([Microsoft Learn][1])

### Multiple Revision Mode

You control the revisions:

```text
v1 → Active
v2 → Active
```

Then you can decide:

```text
100/0
```

then:

```text
80/20
```

then:

```text
50/50
```

then:

```text
0/100
```

This gives you more control over canary and blue-green deployments.

---

# 11. Zero-Downtime Deployment Flow

A good diagram for your students:

```text
                  Production
                      │
                      ▼
                Revision v1
                 nginx-v1
                      │
                   100%
                      │
                      ▼
                    Users


              Deploy Revision v2
                      │
                      ▼
                Revision v2
                 nginx-v2
                      │
                      ▼
                 Start replicas
                      │
                      ▼
                Health checks
                      │
                 ┌────┴────┐
                 │         │
               FAIL       PASS
                 │         │
                 ▼         ▼
              Continue    Ready
              v1          ✓
                           │
                           ▼
                    Switch traffic
                           │
                           ▼
                        v2
```

---

# 12. What If the New Revision Fails?

This is one of the important advantages.

Suppose:

```text
v1 → Healthy
v2 → Failed
```

The existing revision can continue serving traffic while the new revision isn't ready. In single revision mode, traffic doesn't move to the new revision until its readiness conditions are met. ([Microsoft Learn][2])

Conceptually:

```text
v1
 │
 └── 100% traffic

v2
 │
 └── Unhealthy
```

Users continue reaching the existing healthy version.

---

# 13. Zero-Downtime Does Not Mean Zero Risk

This is an important distinction.

Zero-downtime deployment helps avoid an outage **during the transition**, but it doesn't automatically guarantee that the new application version is functionally correct.

For example:

```text
v2
 │
 ├── Container starts ✓
 ├── Readiness probe ✓
 └── Application bug ❌
```

The platform may consider the revision ready if its configured health checks pass, even though an application-level business bug could still exist.

Therefore, for more controlled releases, you can use:

```text
Revision Labels
+
Multiple Revision Mode
+
Traffic Splitting
+
Health Probes
+
Monitoring
```

Azure Monitor can provide metrics, logs, and alerts to compare revisions during deployment. ([Microsoft Learn][6])

---

# 14. Simple Real-World Deployment Strategy

For your course, I would demonstrate it in this sequence:

```text
Step 1
Production
v1 = 100%

        ↓

Step 2
Deploy v2

        ↓

Step 3
v2 starts replicas

        ↓

Step 4
Health/readiness checks

        ↓

Step 5
v2 becomes ready

        ↓

Step 6
Traffic moves to v2

        ↓

Step 7
v1 is deactivated
```

Then teach **canary deployment** separately:

```text
v1 = 80%
v2 = 20%

      ↓

v1 = 50%
v2 = 50%

      ↓

v1 = 0%
v2 = 100%
```

---

## 15. Key Difference

| Concept                      | Purpose                                              |
| ---------------------------- | ---------------------------------------------------- |
| **Revision**                 | Version of application                               |
| **Replica**                  | Running instance of revision                         |
| **Activation**               | Start/stop revision                                  |
| **Traffic Splitting**        | Distribute traffic between revisions                 |
| **Readiness Probe**          | Determine whether replica is ready                   |
| **Zero-Downtime Deployment** | Deploy new version while keeping service available   |
| **Canary Deployment**        | Gradually expose users to new revision               |
| **Blue-Green Deployment**    | Run old/new versions side-by-side and switch traffic |

### Memory Trick

```text
Revision
   ↓
Which version?

Replica
   ↓
How many instances?

Readiness
   ↓
Is it ready?

Traffic
   ↓
Who receives requests?

Zero Downtime
   ↓
Deploy without interrupting users
```

**One-line explanation for your lesson:**

> **Azure Container Apps supports zero-downtime deployment by keeping the existing revision serving traffic until the new revision is successfully provisioned, its replicas are ready, and its health checks pass.** ([Microsoft Learn][2])

[1]: https://learn.microsoft.com/en-us/azure/container-apps/application-lifecycle-management?utm_source=chatgpt.com "Application lifecycle management in Azure Container Apps | Microsoft Learn"
[2]: https://learn.microsoft.com/th-th/Azure/container-apps/revisions?utm_source=chatgpt.com "Update and deploy changes in Azure Container Apps | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/azure/container-apps/revisions-manage?utm_source=chatgpt.com "Manage revisions in Azure Container Apps | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/azure/well-architected/service-guides/azure-container-apps?utm_source=chatgpt.com "Architecture Best Practices for Azure Container Apps - Microsoft Azure Well-Architected Framework | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/Azure/container-apps/connect-apps?utm_source=chatgpt.com "Communicate between container apps in Azure Container Apps | Microsoft Learn"
[6]: https://learn.microsoft.com/en-us/azure/container-apps/observability?utm_source=chatgpt.com "Observability in Azure Container Apps | Microsoft Learn"
