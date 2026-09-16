# Azure Container Apps – Revision Labels

## 1. What is a Revision Label?

A **revision label** is a named pointer to a specific revision of an Azure Container App.

The easiest way to think about it is:

```text
Revision
    ↓
Label
    ↓
Unique URL
```

For example:

```text
myapp--nginx-v1
        ↑
      label: production
```

The label gives you a URL that directly sends traffic to that revision.

Microsoft documents revision labels as unique URLs that can be moved from one revision to another.

---

# 2. Why Do We Need Revision Labels?

Suppose we have:

```text
myapp

Revision 1 → Nginx
Revision 2 → HTTPD
```

We want to test HTTPD without changing production traffic.

Without labels, we might need to change traffic:

```text
Nginx  → 100%
HTTPD  → 0%
```

then:

```text
Nginx  → 90%
HTTPD  → 10%
```

But labels give us another option.

We can create:

```text
production → Nginx
staging    → HTTPD
```

Now:

```text
Production users
       ↓
production URL
       ↓
Nginx


Test users
       ↓
staging URL
       ↓
HTTPD
```

This is one of the most useful purposes of revision labels.

---

# 3. Simple Example

We will use:

```text
Container App:
myapp
```

Revision 1:

```text
myapp--nginx-v1
```

Revision 2:

```text
myapp--httpd-v2
```

We will create:

```text
production → nginx-v1
staging    → httpd-v2
```

Architecture:

```text
                    myapp
                      │
             ┌────────┴────────┐
             │                 │
        production          staging
             │                 │
             ▼                 ▼
        nginx-v1           httpd-v2
             │                 │
             ▼                 ▼
        nginx:alpine       httpd:alpine
```

---

# 4. Revision Label vs Revision Suffix

These two concepts are different.

## Revision Suffix

A suffix gives the revision a meaningful name.

Example:

```text
myapp--nginx-v1
myapp--httpd-v2
```

It answers:

> What is this revision called?

---

## Revision Label

A label points to a revision and provides a dedicated URL.

Example:

```text
production → nginx-v1
staging    → httpd-v2
```

It answers:

> Which revision should this named endpoint point to?

---

# 5. Our Lab

We assume you already have:

```text
Resource Group:
MYRG-India

Environment:
con-env

Container App:
myapp
```

and two revisions:

```text
myapp--nginx-v1
myapp--httpd-v2
```

Check them:

### PowerShell

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

### Bash/Linux

```bash
az containerapp revision list \
  --name myapp \
  --resource-group MYRG-India \
  --all \
  --output table
```

---

# 6. Important Requirement – Multiple Revision Mode

Revision labels are particularly useful with **Multiple revision mode**.

Set the application to Multiple mode:

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

Microsoft documents labels as especially useful when an app is in multiple revision mode.

---

# 7. Activate Both Revisions

Make sure both revisions are active.

For Nginx:

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--nginx-v1
```

For HTTPD:

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2
```

Verify:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You want:

```text
Revision             Active
-------------------  ------
myapp--nginx-v1      True
myapp--httpd-v2      True
```

---

# 8. Create the Production Label

Now we create:

```text
production
```

and point it to:

```text
myapp--nginx-v1
```

### PowerShell

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--nginx-v1 `
  --label production
```

### Bash/Linux

```bash
az containerapp revision label add \
  --name myapp \
  --resource-group MYRG-India \
  --revision myapp--nginx-v1 \
  --label production
```

The current Azure CLI supports `az containerapp revision label add` for assigning a label to a revision.

---

# 9. Create the Staging Label

Now point:

```text
staging
```

to:

```text
myapp--httpd-v2
```

### PowerShell

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --label staging
```

### Bash/Linux

```bash
az containerapp revision label add \
  --name myapp \
  --resource-group MYRG-India \
  --revision myapp--httpd-v2 \
  --label staging
```

Now we have:

```text
production → nginx-v1
staging    → httpd-v2
```

---

# 10. How Does the URL Work?

A revision label gets its own FQDN.

The current documented format is:

```text
<APP_NAME>---<LABEL>.<ENVIRONMENT_UNIQUE_ID>.<REGION>.azurecontainerapps.io
https://myapp---production.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

Notice the **three hyphens (****`---`****)** between the app name and label.

So conceptually:

```text
myapp---production.<environment>.<region>.azurecontainerapps.io
```

and:

```text
myapp---staging.<environment>.<region>.azurecontainerapps.io
https://myapp---staging.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

---

# 11. Production URL

The production label points to:

```text
myapp--nginx-v1
```

So:

```text
Production URL
       ↓
production label
       ↓
Nginx revision
       ↓
nginx:alpine
```

Opening that URL should show:

```text
Welcome to nginx!
```

---

# 12. Staging URL

The staging label points to:

```text
myapp--httpd-v2
```

So:

```text
Staging URL
       ↓
staging label
       ↓
HTTPD revision
       ↓
httpd:alpine
```

Opening that URL should show the Apache HTTP Server default page.

---

# 13. The Most Important Feature – Move the Label

This is where revision labels become very useful.

Suppose:

```text
production → nginx-v1
```

Now we deploy a new application:

```text
httpd-v2
```

We test it through:

```text
staging → httpd-v2
```

After testing, we can move the **production label** to HTTPD.

The URL does not change.

Before:

```text
production
    ↓
nginx-v1
```

After:

```text
production
    ↓
httpd-v2
```

Microsoft specifically documents that labels retain the same URL when moved between revisions.

---

# 14. Move Production Label to HTTPD

Run:

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --label production
```

Now:

```text
production → httpd-v2
```

The production label's URL remains the same, but requests are now directed to the HTTPD revision.

---

# 15. Why Is This Useful?

Imagine your testers have:

```text
https://myapp---staging....
```

and production users have:

```text
https://myapp---production....
```

You don't need to give users a new URL every time you deploy.

You simply move the label.

```text
                    production URL
                          │
                          ▼
                    production
                          │
                    ┌─────┴─────┐
                    │           │
                  v1            v2
                Nginx          HTTPD
```

The label acts like a pointer.

---

# 16. Label as a Pointer

This is the easiest way to remember it.

```text
production
    │
    ▼
Revision 1
```

Then:

```text
production
    │
    ▼
Revision 2
```

The label moved.

The URL stayed the same.

Therefore:

```text
Label = Named pointer to a revision
```

---

# 17. Labels and Traffic Splitting Are Different

This is extremely important.

### Traffic Splitting

Traffic splitting works with the **main Container App URL**.

Example:

```text
Application URL
      │
      ├── 80% → Nginx
      │
      └── 20% → HTTPD
```

### Revision Label

A label gives a dedicated URL for a particular revision.

Example:

```text
staging URL
      │
      ▼
HTTPD
```

Microsoft documents that labels operate independently of traffic splitting.

---

# 18. Simple Comparison

| Feature                    | Traffic Splitting | Revision Label |
| -------------------------- | ----------------- | -------------- |
| Main application URL       | Yes               | No             |
| Dedicated URL              | No                | Yes            |
| 80/20 traffic              | Yes               | No             |
| Directly target revision   | No                | Yes            |
| Useful for testing         | Yes               | Yes            |
| Move URL between revisions | No                | Yes            |
| Stable endpoint            | No                | Yes            |

---

# 19. Traffic Splitting Example

Suppose:

```text
myapp
```

has:

```text
nginx-v1
httpd-v2
```

Traffic splitting:

```text
Main URL
   │
   ├── 80% → nginx-v1
   │
   └── 20% → httpd-v2
```

---

# 20. Revision Label Example

Labels:

```text
production URL
       ↓
nginx-v1

staging URL
       ↓
httpd-v2
```

These are independent of the 80/20 traffic configuration.

---

# 21. Can I Have Both?

Yes.

You can use:

```text
Traffic splitting
+
Revision labels
```

For example:

```text
Main application URL
        │
        ├── 90% → Nginx
        └── 10% → HTTPD


staging URL
        │
        └── HTTPD
```

This gives you a convenient way to test HTTPD directly while still sending only a small percentage of normal application traffic to it.

---

# 22. List Revisions and Labels

You can inspect your revisions:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You should conceptually have:

```text
Revision             Label
-------------------  ----------
myapp--nginx-v1      production
myapp--httpd-v2      staging
```

The exact CLI output can vary with CLI version.

---

# 23. Remove a Label

Suppose you want to remove:

```text
staging
```

Run:

### PowerShell

```powershell
az containerapp revision label remove `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --label staging
```

### Bash/Linux

```bash
az containerapp revision label remove \
  --name myapp \
  --resource-group MYRG-India \
  --revision myapp--httpd-v2 \
  --label staging
```

Microsoft documents `az containerapp revision label remove` for removing a label.

---

# 24. Label Swap

Azure CLI also supports:

```text
az containerapp revision label swap
```

This is useful when you have two labels and want to exchange which revisions they point to.

For example:

Before:

```text
production → Nginx
staging    → HTTPD
```

After swap:

```text
production → HTTPD
staging    → Nginx
```

The Azure CLI currently provides `revision label swap` for this purpose.

This is particularly useful for blue-green deployment patterns.

---

# 25. Blue-Green Example

Let's use our revisions:

```text
Blue
  ↓
nginx-v1

Green
  ↓
httpd-v2
```

Labels:

```text
production → Blue
staging    → Green
```

Architecture:

```text
                 Users
                   │
                   ▼
             production
                   │
                   ▼
                Nginx
                 Blue


             Test Users
                   │
                   ▼
              staging
                   │
                   ▼
                HTTPD
                Green
```

Once Green has been tested:

```text
production → Green
```

Now production uses HTTPD.

---

# 26. Rollback with Labels

Suppose:

```text
production → HTTPD
```

but you want to return to Nginx.

Move the production label back:

```text
production → Nginx
```

The production URL stays unchanged.

This makes label-based rollback easy to demonstrate.

---

# 27. Important Difference from Traffic Rollback

With traffic splitting:

```text
Main URL
   ↓
100% → Nginx
```

With labels:

```text
Production URL
   ↓
Nginx
```

The label approach gives you a **dedicated URL** that points directly to the selected revision.

---

# 28. Complete Hands-on Lab

## Step 1 – Existing revisions

```text
myapp--nginx-v1
myapp--httpd-v2
```

## Step 2 – Multiple mode

```powershell
az containerapp revision set-mode `
  --name myapp `
  --resource-group MYRG-India `
  --mode multiple
```

## Step 3 – Activate both

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--nginx-v1
```

```powershell
az containerapp revision activate `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2
```

## Step 4 – Create production label

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--nginx-v1 `
  --label production
```

## Step 5 – Create staging label

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --label staging
```

---

# 29. Final Architecture

```text
                         myapp
                           │
                ┌──────────┴──────────┐
                │                     │
        myapp--nginx-v1        myapp--httpd-v2
                │                     │
                │                     │
          production               staging
                │                     │
                ▼                     ▼
        Production URL          Staging URL
                │                     │
                ▼                     ▼
             Nginx                  HTTPD
```

---

# 30. Key Commands

### Add label

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision <REVISION_NAME> `
  --label <LABEL_NAME>
```

### Remove label

```powershell
az containerapp revision label remove `
  --name myapp `
  --resource-group MYRG-India `
  --revision <REVISION_NAME> `
  --label <LABEL_NAME>
```

### Swap labels

```powershell
az containerapp revision label swap
```

The exact required arguments for `swap` can be checked with:

```powershell
az containerapp revision label swap --help
```

The current Azure CLI exposes add, remove, and swap operations for revision labels.

---

# 31. Important Notes

### Important Note 1

A revision label is **not the same as a revision suffix**.

```text
Suffix
  ↓
Names the revision

Label
  ↓
Points to the revision and provides a dedicated URL
```

### Important Note 2

A label can point to only one revision at a time.

### Important Note 3

A revision can have only one label at a time.

### Important Note 4

Moving a label changes which revision receives traffic for that label's URL, while the label URL remains the same.

### Important Note 5

Labels and traffic splitting can be used independently or together.

---

# 32. Simple Memory Trick

Remember:

```text
Revision Suffix
      ↓
"What is this revision called?"
```

Example:

```text
myapp--nginx-v1
```

And:

```text
Revision Label
      ↓
"Which revision does this named URL point to?"
```

Example:

```text
production
     ↓
nginx-v1
```

---

# 33. Final Concept

The easiest way to explain Revision Labels is:

> **A revision label is a named pointer with its own URL that points to one specific revision.**

Example:

```text
production
    │
    ▼
nginx-v1
```

Later:

```text
production
    │
    ▼
httpd-v2
```

The **production URL stays the same**; only the revision behind the label changes.

This makes revision labels particularly useful for **testing, staging, blue-green deployments, and controlled promotion of revisions**.
