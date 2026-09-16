Yes. Your screenshot is actually a **very good demonstration of how environment variables and revisions are related**.

You currently have:

```text
Revision             Active   Replicas   Traffic
------------------------------------------------
myapp--v3g5x43       True        0         0%
myapp--0000001       True        0       100%
myapp--0000002       True        0         0%
myapp--0000003       True        0         0%
```

The important point is that **all four revisions are provisioned and healthy, but they currently have 0 replicas**. That can happen when the minimum replica count is 0 and a revision isn't currently receiving traffic. The revision itself still exists.

## 1. Check the environment variables of a revision

You can inspect the configuration of a specific revision with:

```powershell
az containerapp revision show `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--0000001
```

To show only the environment variables:

```powershell
az containerapp revision show `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--0000001 `
  --query "properties.template.containers[0].env" `
  --output table
```

You might get:

```text
Name          Value
------------  ---------
APP_ENV       production
APP_VERSION   v1
```

This tells you **what environment variables are configured for that revision**.

---

# 2. But you asked: "How can I check the variable inside the container?"

That's slightly different.

There are two things we can check:

```text
Revision configuration
        ↓
What variables are configured?

Container
        ↓
What variables does the running container actually see?
```

For the second one, use `az containerapp exec`.

Microsoft provides `az containerapp exec` specifically to connect to a running Container App container. When there are multiple revisions/replicas, you can specify the revision, replica, and container. ([Microsoft Learn][1])

---

# 3. First Find a Running Replica

Your screenshot shows:

```text
Replicas = 0
```

So currently there may be **no running container to exec into**.

First check replicas for your active revision:

```powershell
az containerapp replica list `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--0000001 `
  --output table
```

If you get something like:

```text
Name
-----------------------------------------
myapp--0000001-xxxxxxxxx-xxxxx
```

then you have a running replica.

---

# 4. Get the Container Name

Run:

```powershell
az containerapp replica list `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--0000001 `
  --query "[].{Replica:name,Containers:properties.containers[].name}" `
  --output table
```

You may get:

```text
Replica                              Containers
----------------------------------  ----------
myapp--0000001-xxxxxxxxx-xxxxx      myapp
```

---

# 5. Connect Inside the Container

Now:

```powershell
az containerapp exec `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--0000001 `
  --replica <REPLICA_NAME> `
  --container <CONTAINER_NAME>
```

For example:

```powershell
az containerapp exec `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--0000001 `
  --replica myapp--0000001-xxxxxxxxx-xxxxx `
  --container myapp
```

You'll get a shell inside the container.

---

# 6. Check Environment Variables Inside Linux Container

Once inside:

```bash
env
```

or:

```bash
printenv
```

You might see:

```text
APP_ENV=production
APP_VERSION=v1
HOSTNAME=...
PATH=...
CONTAINER_APP_NAME=myapp
CONTAINER_APP_REVISION=myapp--0000001
```

You can check a specific variable:

```bash
echo $APP_ENV
```

Output:

```text
production
```

Or:

```bash
echo $APP_VERSION
```

Output:

```text
v1
```

---

# 7. Very Useful Built-in Variables

Azure Container Apps also provides built-in environment variables.

For example:

```bash
echo $CONTAINER_APP_NAME
```

Output:

```text
myapp
```

And:

```bash
echo $CONTAINER_APP_REVISION
```

might return:

```text
myapp--0000001
```

There are also built-in variables such as:

```text
CONTAINER_APP_NAME
CONTAINER_APP_REVISION
CONTAINER_APP_HOSTNAME
CONTAINER_APP_PORT
CONTAINER_APP_REPLICA_NAME
```

Microsoft documents these as automatically available runtime variables. ([Microsoft Learn][2])

This gives you a **great demo** because the container itself can identify which revision it is running.

---

# 8. Now the Important Part: Does Every Environment Variable Update Create a Revision?

**Yes — for container environment variables.**

Microsoft's current documentation explicitly says that after a Container App is created, updating its environment variables requires creating a **new revision**. ([Microsoft Learn][2])

For example, you currently have:

```text
myapp--0000001
```

with:

```text
APP_ENV=production
APP_VERSION=v1
```

Now you run:

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --set-env-vars APP_VERSION=v2
```

Azure creates a new revision because the container template changed. ([Microsoft Learn][3])

You could then have:

```text
myapp--0000001
    APP_VERSION=v1

myapp--0000002
    APP_VERSION=v2
```

---

# 9. Let's Demonstrate It With Your `myapp`

I recommend doing this experiment.

### Step 1 – Set variable

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --set-env-vars APP_ENV=production APP_VERSION=v1
```

Then:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You'll see a new revision.

---

### Step 2 – Check its variables

Suppose the new revision is:

```text
myapp--00000004
```

Run:

```powershell
az containerapp revision show `
  --name myapp `
  --resource-group MYRG-India `
  --revision myapp--00000004 `
  --query "properties.template.containers[0].env" `
  --output table
```

You should see:

```text
Name          Value
------------  ----------
APP_ENV       production
APP_VERSION   v1
```

---

# 10. Change Only One Variable

Now:

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --set-env-vars APP_VERSION=v2
```

Then list revisions again:

```powershell
az containerapp revision list `
  --name myapp `
  --resource-group MYRG-India `
  --all `
  --output table
```

You'll have another revision.

Conceptually:

```text
Revision 1
APP_VERSION=v1

        ↓ update

Revision 2
APP_VERSION=v2
```

So yes:

> **Changing a container environment variable creates a new revision.**

---

# 11. Why Your Screenshot Has Four Revisions

Your screenshot:

```text
myapp--v3g5x43
myapp--0000001
myapp--0000002
myapp--0000003
```

is consistent with having made multiple revision-scope changes.

For example:

```text
Initial
   ↓
Revision 1

Change environment variable
   ↓
Revision 2

Change environment variable again
   ↓
Revision 3

Change another container setting
   ↓
Revision 4
```

Azure's revision model creates new revisions when you change the `template`/revision-scope configuration. Environment variables are part of that container template. ([Microsoft Learn][3])

---

# 12. One Important Correction

There is an important distinction with **secrets**.

Normal container environment variable:

```text
APP_VERSION=v2
```

Changing it:

```text
v1 → v2
```

creates a new revision.

But **Container App secret values are application-scoped**. Changing a secret itself doesn't create a new revision; existing active revisions need to be restarted before they recognize the new secret value. ([Microsoft Learn][4])

So:

```text
Environment Variable
       ↓
Revision scoped
       ↓
New revision
```

Whereas:

```text
Secret value
       ↓
Application scoped
       ↓
No new revision
       ↓
Restart revision
```

That's an excellent distinction to teach.

---

# 13. Your Current Screenshot — One More Important Observation

You have:

```text
TrafficWeight
100
```

only on:

```text
myapp--0000001
```

while the others have:

```text
TrafficWeight = 0
```

So currently your traffic configuration is effectively:

```text
myapp--0000001 → 100%
myapp--v3g5x43  → 0%
myapp--0000002  → 0%
myapp--0000003  → 0%
```

And:

```text
Replicas = 0
```

for all of them.

If you want to **exec into a container**, first make sure the revision you're targeting has a running replica. Otherwise `az containerapp exec` has nothing to connect to.

---

## Best Demo for Your Environment Variables Lesson

I recommend we now create a **very small Python/Flask app** that displays:

```text
Application Name
Revision Name
APP_ENV
APP_VERSION
Replica Name
```

Then you can change:

```text
APP_VERSION=v1
```

to:

```text
APP_VERSION=v2
```

and visibly demonstrate:

```text
Environment Variable Change
          ↓
New Revision
          ↓
New Replica
          ↓
Container sees new value
```

That would connect **Environment Variables → Revisions → Replicas → Traffic Splitting → Blue-Green** very nicely.

[1]: https://learn.microsoft.com/en-us/azure/container-apps/container-console?utm_source=chatgpt.com "Connect to a container console in Azure Container Apps | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/azure/container-apps/environment-variables?utm_source=chatgpt.com "Manage environment variables on Azure Container Apps | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/azure/container-apps/revisions-manage?utm_source=chatgpt.com "Manage revisions in Azure Container Apps | Microsoft Learn"
[4]: https://learn.microsoft.com/th-th/Azure/container-apps/revisions?utm_source=chatgpt.com "Update and deploy changes in Azure Container Apps | Microsoft Learn"
