Once your Container App is created, you **don't SSH into the underlying VM/node** like you might with a VM. But you **can open a shell inside the running container** for troubleshooting.

## 1. Using Azure CLI

For your `backend-app`, run:

### PowerShell

```powershell
az containerapp exec `
  --name backend-app `
  --resource-group MYRG-India `
  --command /bin/sh
```

### Bash / Linux

```bash
az containerapp exec \
  --name backend-app \
  --resource-group MYRG-India \
  --command /bin/sh
```

If the container has Bash, you can use:

```powershell
az containerapp exec `
  --name backend-app `
  --resource-group MYRG-India `
  --command /bin/bash
```

But because our Docker image is based on:

```dockerfile
FROM python:3.12-slim
```

`/bin/sh` is the safer choice.

---

# 2. What happens?

You should get an interactive shell similar to:

```text
Starting exec session with pod 'backend-app--xxxxx'...
Connected to container: backend

/app #
```

Now you are **inside the container**.

For example:

```bash
ls
```

You should see:

```text
app.py
```

Check the current directory:

```bash
pwd
```

You should see:

```text
/app
```

Check the running processes:

```bash
ps
```

Depending on the image, the output will show your Python process.

---

# 3. Check Environment Variables

For our frontend application, we created:

```text
BACKEND_URL
```

So you can enter the frontend container:

```powershell
az containerapp exec `
  --name frontend-app `
  --resource-group MYRG-India `
  --command /bin/sh
```

Then:

```bash
echo $BACKEND_URL
```

You should see something like:

```text
https://backend-app.internal.xxxxx.azurecontainerapps.io
```

This is a great way to demonstrate that the frontend container actually received the backend URL.

---

# 4. Test Backend From Inside Frontend

This is especially useful for your demo.

Go inside:

```powershell
az containerapp exec `
  --name frontend-app `
  --resource-group MYRG-India `
  --command /bin/sh
```

Then:

```bash
echo $BACKEND_URL
```

You can also test connectivity if the image contains a suitable HTTP client:

```bash
curl $BACKEND_URL
```

Expected:

```text
Hello from Azure Container Apps Backend!
```

If `curl` isn't installed in the minimal image, you can use Python:

```bash
python -c "import requests; print(requests.get('$BACKEND_URL').text)"
```

This demonstrates:

```text
┌──────────────────────────────┐
│ frontend-app container       │
│                              │
│ echo $BACKEND_URL            │
│          │                   │
│          ↓                   │
│ backend-app                  │
│          │                   │
│          ↓                   │
│ Hello from Backend!          │
└──────────────────────────────┘
```

---

# 5. Azure Portal

You can also use the Azure Portal.

Go to:

**Azure Portal → Container Apps → backend-app**

Then look under the **Console** experience.

Depending on the current portal experience, you can open a console for the running container and execute commands.

Conceptually:

```text
Azure Portal
     │
     ↓
backend-app
     │
     ↓
Console
     │
     ↓
/bin/sh
```

---

# 6. What Are You Actually "Going Inside"?

This is very important.

When you execute:

```bash
az containerapp exec ...
```

you are going inside the **application container**.

You are **NOT** going inside:

```text
Azure VM
```

or:

```text
Kubernetes Node
```

or:

```text
Container Apps Environment
```

The architecture is more like:

```text
Azure Container Apps
│
├── Environment
│
│   ├── frontend-app
│   │      └── Container
│   │
│   └── backend-app
│          └── Container
│
└── Azure-managed infrastructure
```

You can access:

```text
frontend-app
     ↓
Container
     ↓
Shell
```

But you cannot directly manage the underlying Azure infrastructure.

---

# 7. Compare With AKS

This is a good opportunity to understand the difference.

### Container Apps

```text
az containerapp exec
       ↓
Application Container
```

You don't manage the underlying nodes.

### AKS

You have much more Kubernetes-level control:

```text
AKS
│
├── Node
│   └── Pod
│       └── Container
│
├── Node
│   └── Pod
│       └── Container
```

You can use:

```bash
kubectl exec
```

to enter a container.

You can also inspect:

```bash
kubectl get pods
kubectl get nodes
kubectl describe pod
```

With Container Apps, Azure hides that Kubernetes infrastructure.

---

# 8. Useful Commands After Entering the Container

Once you see:

```text
/app #
```

you can run:

```bash
ls
```

```bash
pwd
```

```bash
env
```

```bash
ps
```

```bash
python --version
```

For our backend:

```bash
cat app.py
```

You should see:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Azure Container Apps Backend!"

app.run(host="0.0.0.0", port=5000)
```

---

## 9. Very Important Note About `exec`

`exec` is primarily a **troubleshooting/debugging tool**.

Don't think of it as the normal way you manage your application.

For example, you generally shouldn't do:

```text
exec into container
    ↓
edit application files
    ↓
restart application
```

because those changes are inside the running container and aren't part of your Docker image.

The proper process is:

```text
Change source code
      ↓
Build new Docker image
      ↓
Push new image to ACR
      ↓
Deploy new revision
      ↓
Container Apps runs new version
```

For example:

```text
backend:1.0
     ↓
change code
     ↓
backend:2.0
     ↓
ACR
     ↓
Container App
     ↓
New Revision
```

### Key takeaway

> **`az containerapp exec` allows you to open an interactive shell inside a running Container App container for troubleshooting. You are not accessing the underlying Kubernetes node or VM.**
