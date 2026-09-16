# Azure Container Apps – Revision Suffix / Custom Revision Names

A **revision suffix** allows you to give a revision a meaningful name instead of relying only on Azure's generated revision identifier.

For our simple example, we can use:

```text
myapp--nginx-v1
myapp--httpd-v2
```

This makes revision management much easier to understand.

---

## 1. Why Use a Revision Suffix?

Without a custom suffix, Azure may generate a revision name similar to:

```text
myapp--abc1234
myapp--f7d8k9x
```

These names are difficult to remember.

With a suffix:

```text
myapp--nginx-v1
myapp--httpd-v2
```

we immediately know what each revision contains.

### Our example

```text
Container App
    myapp
       │
       ├── myapp--nginx-v1
       │       │
       │       └── nginx:alpine
       │
       └── myapp--httpd-v2
               │
               └── httpd:alpine
```

---

# 2. Revision Name vs Revision Suffix

This is important.

If the Container App is:

```text
myapp
```

and the revision suffix is:

```text
nginx-v1
```

Azure's revision name is:

```text
myapp--nginx-v1
```

So:

```text
Container App Name + Revision Suffix
                  ↓
        myapp--nginx-v1
```

The `--` separates the Container App name from the revision suffix.

---

# 3. Create Nginx Revision with Custom Suffix

Let's create our first application using:

```text
Image:
nginx:alpine

Revision suffix:
nginx-v1
```

### PowerShell

```powershell
az containerapp create `
  --name myapp `
  --resource-group MYRG-India `
  --environment con-env `
  --image nginx:alpine `
  --target-port 80 `
  --ingress external `
  --revision-suffix nginx-v1
```

### Bash/Linux

```bash
az containerapp create \
  --name myapp \
  --resource-group MYRG-India \
  --environment con-env \
  --image nginx:alpine \
  --target-port 80 \
  --ingress external \
  --revision-suffix nginx-v1
```

The Azure CLI supports `--revision-suffix` when creating a Container App.

---

# 4. Check the Revision

Run:

### PowerShell

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You should see a revision similar to:

```text
Name
--------------------
myapp--nginx-v1
```

Instead of:

```text
myapp--abc123
```

This is much easier to understand.

---

# 5. Create Revision 2 – HTTPD

Now we want to change:

```text
nginx:alpine
```

to:

```text
httpd:alpine
```

and give the new revision the suffix:

```text
httpd-v2
```

### PowerShell

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --image httpd:alpine `
  --revision-suffix httpd-v2
```

### Bash/Linux

```bash
az containerapp update \
  --name myapp \
  --resource-group MYRG-India \
  --image httpd:alpine \
  --revision-suffix httpd-v2
```

Now we should have:

```text
myapp
 │
 ├── myapp--nginx-v1
 │       │
 │       └── nginx:alpine
 │
 └── myapp--httpd-v2
         │
         └── httpd:alpine
```

---

# 6. Why Is This Better?

Compare:

### Without suffix

```text
myapp--abc123
myapp--def456
myapp--xyz789
```

You have to inspect each revision to understand what it contains.

### With suffix

```text
myapp--nginx-v1
myapp--httpd-v2
myapp--nginx-v3
```

You can understand the purpose immediately.

---

# 7. Check Revisions

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
Name                  Active
--------------------  ------
myapp--nginx-v1       False
myapp--httpd-v2       True
```

The exact output can contain additional columns depending on the CLI version.

---

# 8. Revision Suffix Is Not the Docker Image Tag

Don't confuse these two.

### Docker image tag

```text
nginx:alpine
```

Here:

```text
alpine
```

is the Docker image tag.

### ACA revision suffix

```text
nginx-v1
```

Here:

```text
nginx-v1
```

is the Azure Container Apps revision suffix.

Therefore:

```text
Docker Image
    ↓
nginx:alpine

ACA Revision
    ↓
myapp--nginx-v1
```

They are two different concepts.

---

# 9. Example with Real Application Versions

In a real project, you might have:

```text
Docker Images

mycompany/myapp:1.0
mycompany/myapp:2.0
```

and:

```text
ACA Revisions

myapp--v1
myapp--v2
```

Or you could make the names more descriptive:

```text
myapp--release-2026-09
myapp--release-2026-10
```

For a production application, meaningful revision suffixes can make deployment history easier to understand.

---

# 10. Revision Suffix with Canary Deployment

This becomes particularly useful with traffic splitting.

Suppose:

```text
myapp--nginx-v1
        ↓
nginx
```

and:

```text
myapp--httpd-v2
        ↓
httpd
```

Now configure:

```text
myapp--nginx-v1 → 90%
myapp--httpd-v2 → 10%
```

The traffic configuration becomes much easier to read:

```text
              myapp
                │
        ┌───────┴────────┐
        │                │
       90%              10%
        │                │
        ▼                ▼
   nginx-v1           httpd-v2
```

---

# 11. Revision Suffix and Rollback

Suppose HTTPD is receiving:

```text
100%
```

and you want to return to Nginx.

You can refer to the meaningful revision:

```text
myapp--nginx-v1
```

instead of trying to remember:

```text
myapp--abc123
```

For example:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=100
```

Now:

```text
nginx-v1 → 100%
httpd-v2 → 0%
```

---

# 12. Custom Suffix Naming Strategy

For real applications, use a consistent naming convention.

For example:

### Version-based

```text
myapp--v1
myapp--v2
myapp--v3
```

### Environment + version

```text
myapp--prod-v1
myapp--prod-v2
```

### Release-based

```text
myapp--release-101
myapp--release-102
```

### Date-based

```text
myapp--20260916
myapp--20260917
```

For teaching, I recommend keeping it simple:

```text
nginx-v1
httpd-v2
```

---

# 13. Important Restriction

A revision suffix must follow Azure Container Apps naming requirements. It is not an arbitrary unlimited string.

Keep suffixes:

* Short
* Meaningful
* Consistent
* Easy to identify

For example:

```text
nginx-v1
httpd-v2
prod-v3
```

are much easier to manage than very long descriptions.

---

# 14. Portal

When creating a new revision through the Azure Portal, look for the **Revision suffix** field in the revision/container configuration.

For example:

```text
Revision suffix:

nginx-v1
```

When the next revision is deployed:

```text
Revision suffix:

httpd-v2
```

The resulting revisions will be similar to:

```text
myapp--nginx-v1
myapp--httpd-v2
```

The exact location of the field can vary slightly as Microsoft updates the Portal UI.

---

# 15. Simple Hands-on Demonstration

### Step 1

Create:

```text
nginx:alpine
```

with:

```text
revision suffix = nginx-v1
```

Result:

```text
myapp--nginx-v1
```

---

### Step 2

Open the application.

You see:

```text
Welcome to nginx!
```

---

### Step 3

Deploy:

```text
httpd:alpine
```

with:

```text
revision suffix = httpd-v2
```

Result:

```text
myapp--httpd-v2
```

---

### Step 4

List revisions:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You now have:

```text
myapp--nginx-v1
myapp--httpd-v2
```

---

### Step 5

Enable Multiple revision mode.

```powershell
az containerapp revision set-mode `
  --name myapp `
  --resource-group MYRG-India `
  --mode multiple
```

---

### Step 6

Activate both revisions if necessary.

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--nginx-v1
```

---

### Step 7

Split traffic:

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=80 myapp--httpd-v2=20
```

Now:

```text
nginx-v1 → 80%
httpd-v2 → 20%
```

---

# 16. Revision Suffix vs Revision Label

Don't confuse these two.

### Revision suffix

Used to give the revision a meaningful name:

```text
myapp--nginx-v1
```

### Revision label

Used to provide a stable endpoint for accessing a particular revision.

For example, conceptually:

```text
production → nginx-v1
staging    → httpd-v2
```

They solve different problems.

```text
Suffix
  ↓
"What should I call this revision?"

Label
  ↓
"Which revision should this stable endpoint point to?"
```

Revision labels are a separate topic and are especially useful for staging/testing and blue-green deployment scenarios.

---

# 17. Key Commands

### Create with suffix

```powershell
az containerapp create `
  --name myapp `
  --resource-group MYRG-India `
  --environment con-env `
  --image nginx:alpine `
  --target-port 80 `
  --ingress external `
  --revision-suffix nginx-v1
```

### Update with new suffix

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --image httpd:alpine `
  --revision-suffix httpd-v2
```

### List revisions

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

### Set traffic

```powershell
az containerapp ingress traffic set `
  --name myapp `
  --resource-group MYRG-India `
  --revision-weight myapp--nginx-v1=80 myapp--httpd-v2=20
```

---

# 18. Final Concept

Remember:

```text
Container App
     │
     ├── Revision Suffix: nginx-v1
     │       ↓
     │   myapp--nginx-v1
     │       ↓
     │   nginx:alpine
     │
     └── Revision Suffix: httpd-v2
             ↓
         myapp--httpd-v2
             ↓
         httpd:alpine
```

### Simple memory trick

```text
Revision Suffix
       ↓
Meaningful name for a revision
```

For our lab:

```text
nginx-v1
httpd-v2
```

makes revision management much easier than Azure-generated names.

Yes. If you want to **clean up all old revisions** of your Container App, you need to delete them individually. You **cannot delete the currently active revision** while it is serving traffic.

For your example:

```text
myapp
├── myapp--nginx-v1
├── myapp--httpd-v2
└── myapp--v3
```

## 1. First list all revisions

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
myapp--nginx-v1       True
myapp--httpd-v2       False
myapp--v3             False
```

---

## 2. Delete an inactive revision

For example:

```powershell
az containerapp revision delete `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --yes
```

Then:

```powershell
az containerapp revision delete `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--v3 `
  --yes
```

---

# 3. What about the Active Revision?

Suppose:

```text
myapp--nginx-v1 → Active
```

You **should not try to delete it directly** while it is the active revision.

If you want to completely clean up the application, you have two choices.

### Option A — Delete the entire Container App

This is the easiest if this is just your lab:

```powershell
az containerapp delete `
  --name myapp `
  --resource-group MYRG-India `
  --yes
```

This removes the Container App and its revisions.

Your environment `con-env` remains.

```text
MYRG-India
│
├── con-env        ← remains
│
└── myapp          ← deleted
    ├── revision 1
    ├── revision 2
    └── revision 3
```

---

### Option B — Keep the Container App

If you want to keep `myapp`, keep at least one revision active and delete the **inactive revisions** individually.

For example:

```text
myapp
│
├── nginx-v1       ← Active
├── httpd-v2       ← Delete
└── test-v3        ← Delete
```

After cleanup:

```text
myapp
│
└── nginx-v1       ← Active
```

---

# 4. If You Want to Delete Every Revision

If your intention is:

> "I don't need `myapp` anymore; remove everything."

Use:

```powershell
az containerapp delete `
  --name myapp `
  --resource-group MYRG-India `
  --yes
```

That's much simpler than deleting revisions one by one.

---

## Important Note

There is an important distinction:

```text
Delete Revision
       ↓
Removes one revision
```

versus:

```text
Delete Container App
       ↓
Removes the Container App
       ↓
Its revisions are removed with it
```

For your **revision learning lab**, I recommend keeping `myapp` and deleting only the inactive revisions so you can continue experimenting with revision management.

