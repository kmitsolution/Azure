For your previous **frontend + backend** demo, YAML is particularly useful because you can define the Container App configuration in a file.

## 1. Basic Container App YAML

A simple Container App manifest looks like this:

```yaml
properties:
  managedEnvironmentId: /subscriptions/<subscription-id>/resourceGroups/containerapp-rg/providers/Microsoft.App/managedEnvironments/con-env

  configuration:
    ingress:
      external: true
      targetPort: 5000
      transport: auto

  template:
    containers:
      - name: frontend
        image: <acr-login-server>/frontend:1.0
        resources:
          cpu: 0.25
          memory: 0.5Gi
```

Here:

```text
managedEnvironmentId
        ↓
Which Container Apps Environment?

configuration
        ↓
Ingress/network configuration

template
        ↓
Container configuration

containers
        ↓
Container image + resources
```

---

# 2. Create YAML File for Frontend

Create:

```text
frontend.yaml
```

```yaml
properties:
  managedEnvironmentId: /subscriptions/<subscription-id>/resourceGroups/containerapp-rg/providers/Microsoft.App/managedEnvironments/con-env

  configuration:
    ingress:
      external: true
      targetPort: 5000
      transport: auto

  template:
    containers:
      - name: frontend
        image: <acr-login-server>/frontend:1.0

        resources:
          cpu: 0.25
          memory: 0.5Gi

        env:
          - name: BACKEND_URL
            value: https://backend-app.internal.<environment-domain>
```

The important part is:

```yaml
env:
  - name: BACKEND_URL
    value: https://backend-app.internal.<environment-domain>
```

This tells the frontend where the backend is located.

---

# 3. Backend YAML

Create:

```text
backend.yaml
```

```yaml
properties:
  managedEnvironmentId: /subscriptions/<subscription-id>/resourceGroups/containerapp-rg/providers/Microsoft.App/managedEnvironments/con-env

  configuration:
    ingress:
      external: false
      targetPort: 5000
      transport: auto

  template:
    containers:
      - name: backend
        image: <acr-login-server>/backend:1.0

        resources:
          cpu: 0.25
          memory: 0.5Gi
```

Notice:

```yaml
external: false
```

The backend is internal.

So our architecture is:

```text
                    Internet
                       │
                       ↓
                frontend-app
                External Ingress
                       │
                       │ HTTP
                       ↓
                backend-app
                Internal Ingress
```

Both applications:

```text
             con-env
                │
        ┌───────┴────────┐
        ↓                ↓
 frontend-app       backend-app
```

---

# 4. How Do We Deploy the YAML?

First, make sure the Container Apps Environment already exists.

For example:

### Bash / Linux

```bash
az containerapp env create \
  --name con-env \
  --resource-group containerapp-rg \
  --location centralindia
```

### PowerShell

```powershell
az containerapp env create `
  --name con-env `
  --resource-group containerapp-rg `
  --location centralindia
```

Then deploy the backend YAML.

### Bash / Linux

```bash
az containerapp create \
  --name backend-app \
  --resource-group containerapp-rg \
  --yaml backend.yaml
```

### PowerShell

```powershell
az containerapp create `
  --name backend-app `
  --resource-group containerapp-rg `
  --yaml backend.yaml
```

Then deploy the frontend:

### Bash / Linux

```bash
az containerapp create \
  --name frontend-app \
  --resource-group containerapp-rg \
  --yaml frontend.yaml
```

### PowerShell

```powershell
az containerapp create `
  --name frontend-app `
  --resource-group containerapp-rg `
  --yaml frontend.yaml
```

---

# 5. Important: ACR Authentication

If the ACR is private, Container Apps needs permission to pull the image.

For a simple demonstration, you can configure registry credentials.

The YAML can contain a `registries` section:

```yaml
properties:
  managedEnvironmentId: /subscriptions/<subscription-id>/resourceGroups/containerapp-rg/providers/Microsoft.App/managedEnvironments/con-env

  configuration:

    registries:
      - server: <acr-login-server>
        username: <acr-username>
        passwordSecretRef: acr-password

    secrets:
      - name: acr-password
        value: <acr-password>

    ingress:
      external: true
      targetPort: 5000
      transport: auto

  template:
    containers:
      - name: frontend
        image: <acr-login-server>/frontend:1.0

        resources:
          cpu: 0.25
          memory: 0.5Gi
```

However, **don't use this approach for production**, because you're putting the registry authentication configuration into the YAML.

A better production architecture is:

```text
Container App
      │
      │ Managed Identity
      ↓
Azure Container Registry
      │
      │ AcrPull
      ↓
Docker Image
```

---

# 6. Get Environment Information

You can get the Container Apps Environment resource ID using:

### Bash / Linux

```bash
az containerapp env show \
  --name con-env \
  --resource-group containerapp-rg \
  --query id \
  --output tsv
```

### PowerShell

```powershell
az containerapp env show `
  --name con-env `
  --resource-group containerapp-rg `
  --query id `
  --output tsv
```

You will get something similar to:

```text
/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/containerapp-rg/providers/Microsoft.App/managedEnvironments/con-env
```

Put that value into:

```yaml
managedEnvironmentId:
```

---

# 7. Important Difference: YAML vs CLI Parameters

Previously we created the application like this:

```bash
az containerapp create \
  --name frontend-app \
  --resource-group containerapp-rg \
  --environment con-env \
  --image myacr.azurecr.io/frontend:1.0 \
  --target-port 5000 \
  --ingress external
```

With YAML:

```bash
az containerapp create \
  --name frontend-app \
  --resource-group containerapp-rg \
  --yaml frontend.yaml
```

The configuration moves from the CLI command into the YAML file.

### CLI approach

```text
CLI command
   │
   ├── image
   ├── ingress
   ├── port
   ├── environment
   └── resources
```

### YAML approach

```text
frontend.yaml
   │
   ├── environment
   ├── image
   ├── ingress
   ├── port
   ├── resources
   └── environment variables
```

This becomes much easier to manage when the application configuration grows.

---

# 8. Verify the Container App

Check the frontend:

### Bash / Linux

```bash
az containerapp show \
  --name frontend-app \
  --resource-group containerapp-rg \
  --query properties.configuration.ingress.fqdn \
  --output tsv
```

### PowerShell

```powershell
az containerapp show `
  --name frontend-app `
  --resource-group containerapp-rg `
  --query properties.configuration.ingress.fqdn `
  --output tsv
```

Then open:

```text
https://<frontend-fqdn>
```

The frontend should call:

```text
frontend-app
     ↓
BACKEND_URL
     ↓
backend-app
```

and display:

```text
Frontend Container App

Response from Backend:

Hello from Azure Container Apps Backend!
```

---

# 9. YAML Mental Model

For AZ-104, remember the basic structure:

```yaml
properties:

  managedEnvironmentId:
      ↓
  Container Apps Environment

  configuration:
      ↓
  Ingress
  Registry
  Secrets
  Other configuration

  template:
      ↓
  Application template

  containers:
      ↓
  Container image
  CPU
  Memory
  Environment variables
```

So:

> **The YAML manifest describes the desired configuration of the Container App, while Azure Container Apps manages the underlying infrastructure.**

---

## Important Note

For your learning sequence, I recommend demonstrating YAML in this order:

```text
Demo 1
Quickstart Container App
        ↓
Portal

Demo 2
Frontend + Backend
        ↓
Dockerfile
        ↓
Docker Image
        ↓
ACR
        ↓
Container Apps

Demo 3
Frontend + Backend
        ↓
YAML Manifest
        ↓
az containerapp create --yaml
```

This makes it very clear that **YAML is simply another way to define the Container App configuration**, rather than a different hosting technology.
