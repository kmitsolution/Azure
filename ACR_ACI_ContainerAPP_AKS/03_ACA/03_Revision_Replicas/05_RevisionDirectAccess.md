**Direct Revision Access** in Azure Container Apps means accessing a **specific revision directly**, instead of relying on the main application URL and traffic-splitting rules.

Microsoft documents this as a use case for revision labels: a label gives a URL that routes directly to one specific revision. ([Microsoft Learn][1])

## 1. Your Example

You currently have:

```text
Container App:
myapp

Revision 1:
myapp--nginx-v1

Revision 2:
myapp--httpd-v2
```

And your main URL is:

```text
https://myapp.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

If traffic splitting is configured, the main URL could send requests to either revision.

For **direct revision access**, we create labels:

```text
production → myapp--nginx-v1

staging → myapp--httpd-v2
```

Then:

```text
Production URL
       ↓
production label
       ↓
myapp--nginx-v1
       ↓
Nginx
```

and:

```text
Staging URL
       ↓
staging label
       ↓
myapp--httpd-v2
       ↓
HTTPD
```

---

# 2. Why Direct Revision Access?

Suppose your main application is:

```text
myapp
```

with:

```text
Revision 1 → Nginx
Revision 2 → HTTPD
```

If the main URL has:

```text
Revision 1 → 80%
Revision 2 → 20%
```

you cannot use the main URL to guarantee that your request reaches HTTPD.

With direct revision access:

```text
staging URL → HTTPD
```

Every request to that staging URL goes to the specific revision associated with the label. ([Microsoft Learn][2])

---

# 3. Create Direct Access for Nginx

First make sure the revision exists:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You should see something like:

```text
Name
------------------
myapp--nginx-v1
myapp--httpd-v2
```

Now assign the `production` label:

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--nginx-v1 `
  --label production
```

---

# 4. Create Direct Access for HTTPD

Assign the `staging` label:

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --label staging
```

Now:

```text
production → Nginx

staging → HTTPD
```

---

# 5. URLs

Your environment domain is:

```text
icyfield-fdc08e5e.centralindia.azurecontainerapps.io
```

The label FQDN format is:

```text
<APP_NAME>---<LABEL>.<ENVIRONMENT_UNIQUE_ID>.<REGION>.azurecontainerapps.io
```

Notice the **three hyphens `---`** between the app name and label. ([Microsoft Learn][2])

Therefore:

### Production

```text
https://myapp---production.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

### Staging

```text
https://myapp---staging.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

---

# 6. Main URL vs Direct Revision URL

This distinction is important for your students.

### Main URL

```text
https://myapp.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

Traffic may be:

```text
                Main URL
                   │
          ┌────────┴────────┐
          │                 │
        80%               20%
          │                 │
          ▼                 ▼
       Nginx              HTTPD
```

### Direct Revision URL

```text
https://myapp---staging.icyfield-fdc08e5e.centralindia.azurecontainerapps.io/
```

Traffic:

```text
             Staging URL
                  │
                  ▼
             staging label
                  │
                  ▼
           myapp--httpd-v2
                  │
                  ▼
                HTTPD
```

So the staging URL is specifically associated with the HTTPD revision.

---

# 7. Very Important: Label ≠ Revision

The label is **not** the revision itself.

Think:

```text
Revision
   ↓
myapp--httpd-v2

Label
   ↓
staging
```

Relationship:

```text
staging
   │
   ▼
myapp--httpd-v2
```

The label provides the direct-access URL.

---

# 8. Direct Access Is Excellent for Testing

Imagine you deploy:

```text
Revision 1 → Production
Revision 2 → New Version
```

You don't want all production users to test Revision 2.

You can create:

```text
staging → Revision 2
```

Then your testers use:

```text
https://myapp---staging....
```

while normal users continue using:

```text
https://myapp....
```

This is one of the practical uses Microsoft identifies for revision labels. ([Microsoft Learn][2])

---

# 9. Move Direct Access to Another Revision

This is particularly powerful.

Initially:

```text
production
     ↓
nginx-v1
```

Later:

```text
production
     ↓
httpd-v2
```

Run:

```powershell
az containerapp revision label add `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--httpd-v2 `
  --label production
```

Now the same production label URL points to HTTPD.

The URL doesn't change; the revision behind the label changes. ([Microsoft Learn][3])

---

# 10. Direct Revision Access vs Traffic Splitting

| Feature                           | Direct Revision Access | Traffic Splitting   |
| --------------------------------- | ---------------------- | ------------------- |
| Uses revision labels              | Yes                    | No                  |
| Dedicated URL                     | Yes                    | No                  |
| Specific revision                 | Yes                    | Percentage-based    |
| Example                           | `staging → HTTPD`      | `80% / 20%`         |
| Good for testing                  | Yes                    | Yes                 |
| A/B testing                       | Possible               | Yes                 |
| Main app URL                      | Not required           | Yes                 |
| Move endpoint to another revision | Yes                    | Change traffic rule |

Microsoft notes that labels and traffic splitting can be used independently or together. ([Microsoft Learn][1])

---

## Simple Memory Trick

Teach it like this:

```text
Main URL
   ↓
"Give me the application"

Label URL
   ↓
"Give me THIS revision"
```

Or:

```text
Traffic Splitting
      ↓
WHO gets traffic?

Revision Label
      ↓
WHICH revision do I want directly?
```

**Direct Revision Access = Revision Label + Dedicated URL.**

[1]: https://learn.microsoft.com/th-th/Azure/container-apps/revisions?utm_source=chatgpt.com "Update and deploy changes in Azure Container Apps | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/Azure/container-apps/connect-apps?utm_source=chatgpt.com "Communicate between container apps in Azure Container Apps | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/azure/container-apps/revisions-manage?utm_source=chatgpt.com "Manage revisions in Azure Container Apps | Microsoft Learn"
