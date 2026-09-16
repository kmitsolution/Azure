# Azure Container Apps – Revisions and Revision Management

## Hands-on Demo Using Nginx and HTTPD from Docker Hub

---

# 1. Objective

In this lab, we will understand **Azure Container Apps Revisions** using two very simple Docker Hub images:

```text
nginx:alpine
httpd:alpine
```

We will deploy:

```text
Revision 1 → Nginx
Revision 2 → HTTPD
```

Then we will learn how to:

* Create a Container App
* Understand revisions
* Create a new revision
* Use Single Revision mode
* Use Multiple Revision mode
* Activate and deactivate revisions
* Split traffic between revisions
* Send 100% traffic to one revision
* Roll back traffic to an older revision
* Delete an old revision
* Understand Revision vs Replica

Azure Container Apps automatically creates the first revision when a Container App is deployed. New revisions are created when revision-scope changes are deployed.

---

# 2. What is a Revision?

A **revision** is a specific version of a Container App.

For example:

```text
Container App
     │
     ├── Revision 1 → Nginx
     │
     └── Revision 2 → HTTPD
```

Think of a revision as:

```text
Revision = Version of the application
```

In our example:

```text
Revision 1
    ↓
nginx:alpine

Revision 2
    ↓
httpd:alpine
```

Both are versions of the same Container App.

---

# 3. Our Simple Architecture

We will use one Container App:

```text
Container App
    myapp
       │
       ├── Revision 1
       │      │
       │      └── nginx:alpine
       │
       └── Revision 2
              │
              └── httpd:alpine
```

Both images listen on:

```text
Port 80
```

This makes the demonstration very simple.

---

# 4. Why Nginx and HTTPD?

We could use a custom application, but that would introduce unnecessary complexity.

Instead:

```text
Nginx
Docker Hub:
nginx:alpine
```

and:

```text
Apache HTTP Server
Docker Hub:
httpd:alpine
```

are enough.

When we access the application:

### Nginx revision

We will see the Nginx default page:

```text
Welcome to nginx!
```

### HTTPD revision

We will see the Apache HTTP Server default page:

```text
It works!
```

Therefore, it becomes very easy to understand which revision is receiving the request.

---

# 5. Lab Resources

We will use:

| Resource                   | Value          |
| -------------------------- | -------------- |
| Resource Group             | `MYRG-India`   |
| Region                     | `centralindia` |
| Container Apps Environment | `con-env`      |
| Container App              | `myapp`        |
| Revision 1                 | Nginx          |
| Revision 2                 | HTTPD          |
| Nginx Image                | `nginx:alpine` |
| HTTPD Image                | `httpd:alpine` |
| Target Port                | `80`           |
| Ingress                    | External       |

---

# 6. Revision Modes

Azure Container Apps has two main active revision modes:

```text
Single
Multiple
```

The default revision mode is **Single**.

---

## Single Revision Mode

Only one revision is active at a time.

Conceptually:

```text
Container App
     │
     └── Revision 1
             │
           Nginx
             │
           100%
```

After deploying a new revision:

```text
Container App
     │
     └── Revision 2
             │
           HTTPD
             │
           100%
```

The previous revision is no longer active.

---

## Multiple Revision Mode

Multiple revisions can be active at the same time.

For example:

```text
Revision 1 → Nginx  → 80%
Revision 2 → HTTPD  → 20%
```

Azure Container Apps can split incoming traffic between active revisions according to configured traffic weights.

---

# 7. Step 1 – Create Resource Group

If you already have the resource group, skip this step.

### PowerShell

```powershell
az group create `
  --name MYRG-India `
  --location centralindia
```

### Bash/Linux

```bash
az group create \
  --name MYRG-India \
  --location centralindia
```

---

# 8. Step 2 – Create Container Apps Environment

If you already have `con-env`, skip this step.

### PowerShell

```powershell
az containerapp env create `
  --name con-env `
  --resource-group MYRG-India `
  --location centralindia
```

### Bash/Linux

```bash
az containerapp env create \
  --name con-env \
  --resource-group MYRG-India \
  --location centralindia
```

---

# 9. Step 3 – Create the First Container App Using Nginx

We will use:

```text
Docker Hub Image:
nginx:alpine
```

The Nginx container listens on:

```text
80
```

### PowerShell

```powershell
az containerapp create `
  --name myapp `
  --resource-group MYRG-India `
  --environment con-env `
  --image nginx:alpine `
  --target-port 80 `
  --ingress external
```

### Bash/Linux

```bash
az containerapp create \
  --name myapp \
  --resource-group MYRG-India \
  --environment con-env \
  --image nginx:alpine \
  --target-port 80 \
  --ingress external
```

Azure creates the first revision automatically when the Container App is deployed.

---

# 10. Step 4 – Get the Application URL

### PowerShell

```powershell
$FQDN = az containerapp show `
  --name myapp `
  --resource-group MYRG-India `
  --query properties.configuration.ingress.fqdn `
  --output tsv

$FQDN
```

### Bash/Linux

```bash
FQDN=$(az containerapp show \
  --name myapp \
  --resource-group MYRG-India \
  --query properties.configuration.ingress.fqdn \
  --output tsv)

echo $FQDN
```

You will get something similar to:

```text
myapp.xxxxxx.centralindia.azurecontainerapps.io
```

Open:

```text
https://<FQDN>
```

---

# 11. Step 5 – Verify Nginx

The browser should display the Nginx default page.

You should see something similar to:

```text
Welcome to nginx!
```

This tells us:

```text
Request
   ↓
Container App
   ↓
Nginx Revision
   ↓
nginx:alpine
```

---

# 12. Step 6 – List Revisions

Run:

### PowerShell

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --output table
```

### Bash/Linux

```bash
az containerapp revision list \
  --name myapp \
  --resource-group MYRG-India \
  --output table
```

The current Azure CLI provides `az containerapp revision list` for listing revisions.

You should see one revision.

For example:

```text
Name                  Active    Replicas
--------------------  --------  --------
myapp--xxxxxxxx       True      1
```

The exact revision name will be generated by Azure.

---

# 13. Understanding Our First Revision

At this point:

```text
Container App
     │
     ▼
   myapp
     │
     ▼
Revision 1
     │
     ▼
nginx:alpine
     │
     ▼
Port 80
```

Traffic:

```text
Revision 1 → 100%
```

---

# 14. Step 7 – Create Revision 2 Using HTTPD

Now we will change the container image:

```text
nginx:alpine
```

to:

```text
httpd:alpine
```

This is a **revision-scope change**, so deploying the change creates a new revision.

### PowerShell

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --image httpd:alpine
```

### Bash/Linux

```bash
az containerapp update \
  --name myapp \
  --resource-group MYRG-India \
  --image httpd:alpine
```

---

# 15. What Happened?

Before:

```text
Revision 1
     │
     ▼
nginx:alpine
```

After the update:

```text
Revision 2
     │
     ▼
httpd:alpine
```

Azure has created a new revision because we changed the container image.

---

# 16. List Revisions Again

Run:

### PowerShell

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --output table
```

You should now see the new revision.

In Single mode, the newer revision becomes the active revision while the previous revision is no longer active.

Conceptually:

```text
Revision 1 → Nginx   → Inactive
Revision 2 → HTTPD   → Active
```

---

# 17. Test the Application Again

Open:

```text
https://<FQDN>
```

Now you should see the Apache HTTP Server default page.

Conceptually:

```text
Browser
   │
   ▼
Container App
   │
   ▼
Revision 2
   │
   ▼
httpd:alpine
```

This is our first demonstration of revision management.

---

# 18. What Changed?

We did not create another Container App.

We still have:

```text
Container App:
myapp
```

But now it has multiple revisions:

```text
myapp
 │
 ├── Revision 1 → nginx:alpine
 │
 └── Revision 2 → httpd:alpine
```

This is the key concept.

```text
Container App ≠ Revision
```

A Container App can have multiple revisions.

---

# 19. Revision vs Container App

Think of:

```text
Container App
```

as the application name.

And:

```text
Revision
```

as a version of that application.

Example:

```text
myapp
 │
 ├── v1 → Nginx
 │
 ├── v2 → HTTPD
 │
 └── v3 → another image
```

---

# 20. Step 8 – Enable Multiple Revision Mode

To demonstrate traffic splitting, we need multiple active revisions.

Set the revision mode to:

```text
Multiple
```

### PowerShell

```powershell
az containerapp revision set-mode `
  --name myapp `
  --resource-group MYRG-India `
  --mode multiple
```

### Bash/Linux

```bash
az containerapp revision set-mode \
  --name myapp \
  --resource-group MYRG-India \
  --mode multiple
```

Azure CLI supports `single` and `multiple` as the revision mode values. Changing revision mode itself does not create a new revision.

---

# 21. Why Multiple Mode?

We want:

```text
Revision 1
Nginx
```

and:

```text
Revision 2
HTTPD
```

to both be active.

Then we can control traffic:

```text
Nginx  → 80%
HTTPD  → 20%
```

---

# 22. Find the Revision Names

Run:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You might see:

```text
Name                  Active
--------------------  ------
myapp--abc123         True
myapp--def456         True
```

The actual names will be different in your environment.

For the rest of the lab, suppose:

```text
Nginx Revision:
myapp--abc123

HTTPD Revision:
myapp--def456
```

Replace these names with your actual revision names.

---

# 23. Activate an Old Revision

If the Nginx revision is inactive, activate it.

### PowerShell

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--abc123
```

### Bash/Linux

```bash
az containerapp revision activate \
  --name myapp \
  --resource-group MYRG-India \
  --revision myapp--abc123
```

Azure provides `activate` and `deactivate` commands for revision management.

---

# 24. Verify Both Revisions

Run:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

Conceptually:

```text
Revision             Image          Active
-------------------  -------------  ------
myapp--abc123        nginx:alpine   True
myapp--def456        httpd:alpine   True
```

Now both revisions are active.

---

# 25. Step 9 – Traffic Splitting

Now we can split traffic.

For example:

```text
Nginx → 80%
HTTPD → 20%
```

Use:

### PowerShell

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--abc123=80 myapp--def456=20
```

### Bash/Linux

```bash
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MYRG-India \
  --revision-weight myapp--abc123=80 myapp--def456=20
```

The current Azure CLI supports `--revision-weight` for assigning traffic percentages to revisions.

---

# 26. Traffic Architecture

Now the architecture is:

```text
                       Users
                         │
                         ▼
                  Container App
                      myapp
                         │
                ┌────────┴────────┐
                │                 │
               80%               20%
                │                 │
                ▼                 ▼
           Revision 1         Revision 2
              Nginx              HTTPD
                │                 │
                ▼                 ▼
          nginx:alpine       httpd:alpine
```

---

# 27. What Will Happen in the Browser?

When you repeatedly refresh the application:

```text
https://<FQDN>
```

requests are distributed according to the configured weights.

Approximately:

```text
80% → Nginx
20% → HTTPD
```

Because the two default pages look different, this makes traffic splitting easy to demonstrate.

The exact sequence is not guaranteed to alternate `Nginx → HTTPD → Nginx`; the percentages describe traffic distribution rather than a fixed request-by-request rotation.

---

# 28. View Traffic Configuration

Run:

### PowerShell

```powershell
az containerapp ingress traffic show `
  --name myapp `
  --resource-group MYRG-India `
  --output table
```

You should see the configured traffic allocation.

---

# 29. Step 10 – Move 100% Traffic to HTTPD

Suppose we have tested HTTPD and now want:

```text
HTTPD → 100%
Nginx → 0%
```

Run:

### PowerShell

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--def456=100
```

### Bash/Linux

```bash
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MYRG-India \
  --revision-weight myapp--def456=100
```

Now:

```text
Users
  │
  ▼
HTTPD Revision
  │
  ▼
100%
```

---

# 30. Step 11 – Roll Back to Nginx

Suppose HTTPD is the new version but you want to return traffic to Nginx.

We don't need to rebuild Nginx.

We can simply route traffic back to the Nginx revision.

```text
Nginx → 100%
HTTPD → 0%
```

### PowerShell

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--abc123=100
```

### Bash/Linux

```bash
az containerapp ingress traffic set \
  --name myapp \
  --resource-group MYRG-India \
  --revision-weight myapp--abc123=100
```

This demonstrates a simple rollback using an existing revision.

---

# 31. Revision Rollback Concept

The important idea is:

```text
Before

Nginx → 0%
HTTPD → 100%
```

Rollback:

```text
Nginx → 100%
HTTPD → 0%
```

Architecture:

```text
                 Container App
                      │
              Traffic Management
                      │
              ┌───────┴───────┐
              │               │
           Revision 1      Revision 2
             Nginx            HTTPD
              100%              0%
```

---

# 32. Step 12 – Deactivate a Revision

Suppose we no longer want the HTTPD revision running.

First make sure it has no traffic:

```text
HTTPD → 0%
```

Then deactivate it.

### PowerShell

```powershell
az containerapp revision deactivate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--def456
```

### Bash/Linux

```bash
az containerapp revision deactivate \
  --name myapp \
  --resource-group MYRG-India \
  --revision myapp--def456
```

Deactivating a revision stops the running replicas for that revision.

---

# 33. Active vs Inactive Revision

After deactivation:

```text
Revision             Image          Status
-------------------  -------------  --------
myapp--abc123        nginx:alpine   Active
myapp--def456        httpd:alpine   Inactive
```

The inactive revision is retained as a revision record but isn't serving traffic.

---

# 34. Step 13 – Delete a Revision

If you are finished with a revision, you can delete it.

First list the revisions:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

Then delete the unwanted revision:

```powershell
az containerapp revision delete `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--def456 `
  --yes
```

Use deletion carefully. If you want the revision available for rollback, keep it instead of deleting it.

---

# 35. Portal – View Revisions

You can perform revision management from the Azure Portal.

Open:

```text
Azure Portal
   ↓
Container Apps
   ↓
myapp
```

Under:

```text
Application
```

select:

```text
Revisions and replicas
```

Depending on the current Portal UI, you may also see revision management options directly in the application navigation.

Microsoft's current documentation uses the revision management area to view revisions, create revisions, select revision mode, and manage traffic.

---

# 36. Portal – View Nginx Revision

You should see something similar to:

```text
Revision             Status
-------------------  --------
myapp--abc123        Active
```

The revision details will show the container configuration.

You can inspect the image:

```text
nginx:alpine
```

---

# 37. Portal – Create HTTPD Revision

In the Container App, create a new revision.

Change:

```text
Container Image
```

from:

```text
nginx:alpine
```

to:

```text
httpd:alpine
```

Keep:

```text
Target Port:
80
```

because both containers listen on port 80.

Deploy the new revision.

You should now have:

```text
Revision 1
nginx:alpine

Revision 2
httpd:alpine
```

---

# 38. Portal – Change Revision Mode

In the revision management area, choose:

```text
Multiple
```

This allows multiple revisions to remain active.

Conceptually:

```text
Single:

Revision 1 → Active
Revision 2 → Inactive
```

versus:

```text
Multiple:

Revision 1 → Active
Revision 2 → Active
```

Multiple mode is required when you want to split traffic between active revisions.

---

# 39. Portal – Traffic Splitting

In Revision Management, configure traffic.

Example:

```text
Revision             Traffic
-------------------  -------
Nginx                80%
HTTPD                20%
```

Apply the configuration.

Then test the application URL.

---

# 40. Portal – Move Traffic to HTTPD

Change:

```text
Nginx → 0%
HTTPD → 100%
```

Apply.

Now all normal application traffic goes to HTTPD.

---

# 41. Portal – Roll Back

Change:

```text
Nginx → 100%
HTTPD → 0%
```

Apply.

The Nginx revision becomes the traffic destination again.

---

# 42. Revision Management Commands

## List revisions

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

---

## Show revision

```powershell
az containerapp revision show `
  --name myapp `
  --resource-group MYRG-India `
  --revision <REVISION_NAME>
```

---

## Activate revision

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision <REVISION_NAME>
```

---

## Deactivate revision

```powershell
az containerapp revision deactivate `
  --name myapp `
  --resource-group MYRG-India `
  --revision <REVISION_NAME>
```

---

## Restart revision

```powershell
az containerapp revision restart `
  --name myapp `
  --resource-group MYRG-India `
  --revision <REVISION_NAME>
```

---

## Set revision mode

```powershell
az containerapp revision set-mode `
  --name myapp `
  --resource-group MYRG-India `
  --mode multiple
```

---

## Set traffic

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight <REVISION1>=80 <REVISION2>=20
```

---

## Show traffic

```powershell
az containerapp ingress traffic show `
  --name myapp `
  --resource-group MYRG-India
```

---

# 43. Revision vs Replica

This is another important concept.

## Revision

A version of the application.

```text
Revision 1
   ↓
Nginx
```

## Replica

A running instance of that revision.

For example:

```text
Revision 1
   │
   ├── Replica 1
   ├── Replica 2
   └── Replica 3
```

So:

```text
Revision = Version
Replica = Running instance
```

---

# 44. Example – Multiple Replicas

Suppose:

```text
Revision 1
Image: nginx:alpine
```

and scaling creates 3 replicas:

```text
Revision 1
     │
     ├── Nginx Replica 1
     ├── Nginx Replica 2
     └── Nginx Replica 3
```

Now suppose Revision 2 uses HTTPD:

```text
Revision 2
     │
     ├── HTTPD Replica 1
     └── HTTPD Replica 2
```

Architecture:

```text
Container App
│
├── Revision 1
│     │
│     ├── Replica 1
│     ├── Replica 2
│     └── Replica 3
│
└── Revision 2
      │
      ├── Replica 1
      └── Replica 2
```

---

# 45. Complete Hands-on Flow

The complete lab can be remembered as:

```text
STEP 1
Create Container App
        │
        ▼
nginx:alpine
        │
        ▼
Revision 1
        │
        ▼
100% Traffic
        │
        ▼
Test Nginx
        │
        ▼
Update Image
        │
        ▼
httpd:alpine
        │
        ▼
Revision 2
        │
        ▼
100% Traffic to HTTPD
        │
        ▼
Enable Multiple Mode
        │
        ▼
Activate Both Revisions
        │
        ▼
80% Nginx / 20% HTTPD
        │
        ▼
Test Traffic Splitting
        │
        ▼
100% HTTPD
        │
        ▼
Rollback
        │
        ▼
100% Nginx
        │
        ▼
Deactivate HTTPD
        │
        ▼
Delete HTTPD Revision
```

---

# 46. Real-World Mapping

Our simple lab uses:

```text
Revision 1 → Nginx
Revision 2 → HTTPD
```

In a real application, the same concept could be:

```text
Revision 1 → myapp:v1
Revision 2 → myapp:v2
```

For example:

```text
Revision 1
mycompany/myapp:1.0

Revision 2
mycompany/myapp:2.0
```

The revision mechanism is exactly what we are demonstrating with Nginx and HTTPD.

---

# 47. Canary Deployment Example

Suppose:

```text
Revision 1 → Application v1
Revision 2 → Application v2
```

Instead of immediately moving everyone to v2:

```text
v1 → 90%
v2 → 10%
```

Test v2.

Then:

```text
v1 → 50%
v2 → 50%
```

Finally:

```text
v1 → 0%
v2 → 100%
```

This is a typical progressive traffic rollout using multiple active revisions.

---

# 48. Blue/Green Deployment Example

Another common pattern:

```text
Blue
Revision 1
v1
```

```text
Green
Revision 2
v2
```

Initially:

```text
Blue → 100%
Green → 0%
```

After testing:

```text
Blue → 0%
Green → 100%
```

The important point is that the traffic is moved between revisions rather than creating a completely separate Container App.

---

# 49. Key Concepts to Remember

### Concept 1

```text
Container App
```

is the application.

### Concept 2

```text
Revision
```

is a version of the application.

### Concept 3

```text
Replica
```

is a running instance of a revision.

### Concept 4

```text
Single Mode
```

allows one active revision.

### Concept 5

```text
Multiple Mode
```

allows multiple active revisions.

### Concept 6

Traffic can be distributed between active revisions in Multiple mode.

### Concept 7

A revision can be activated or deactivated.

### Concept 8

An older revision can be used for rollback if it is still available.

---

# 50. Simple Memory Trick

Remember:

```text
Container App
      ↓
   Revision
      ↓
 Application Version
      ↓
   Replicas
      ↓
 Running Instances
```

And:

```text
Single
   ↓
One active revision

Multiple
   ↓
Multiple active revisions
   ↓
Traffic splitting
```

---

# 51. Final Architecture

```text
                         Internet
                            │
                            ▼
                    Azure Container App
                           myapp
                            │
                     Traffic Management
                            │
              ┌─────────────┴─────────────┐
              │                           │
            80%                          20%
              │                           │
              ▼                           ▼
       Revision 1                   Revision 2
          Nginx                       HTTPD
              │                           │
              ▼                           ▼
       nginx:alpine                 httpd:alpine
              │                           │
              ▼                           ▼
          Port 80                     Port 80
```

---

# 52. Final Takeaway

The easiest way to understand Azure Container Apps revisions is:

```text
Container App = Application
Revision      = Version
Replica       = Running instance
```

Our lab demonstrates:

```text
Revision 1
     ↓
Nginx

Revision 2
     ↓
HTTPD
```

Then:

```text
Single Mode
     ↓
One active revision
```

and:

```text
Multiple Mode
     ↓
Multiple active revisions
     ↓
Traffic splitting
     ↓
Canary / Blue-Green style deployments
     ↓
Rollback
```

The complete deployment lifecycle is:

```text
Deploy Nginx
     ↓
Revision 1
     ↓
Deploy HTTPD
     ↓
Revision 2
     ↓
Activate both
     ↓
Split traffic
     ↓
Test
     ↓
Move 100% traffic
     ↓
Rollback if required
     ↓
Deactivate old revision
     ↓
Delete old revision
```

This simple **Nginx → HTTPD** example provides the foundation for understanding how the same revision-management mechanism is used with real application versions such as:

```text
myapp:v1
myapp:v2
myapp:v3
```

Microsoft's current documentation confirms that revision-scope changes create new revisions, while revision mode and traffic configuration are application-level settings.
