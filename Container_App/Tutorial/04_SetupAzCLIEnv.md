# Setup Azure Environment for Azure Container Apps Using Azure CLI (Step by Step)

This guide assumes you are using **Windows 10/11**.

---

# Step 1: Install Azure CLI

Download and install Azure CLI from:

[Azure CLI Installation Documentation](https://learn.microsoft.com/cli/azure/install-azure-cli-windows?utm_source=chatgpt.com)

Or install using PowerShell:

```powershell
winget install Microsoft.AzureCLI
```

Verify installation:

```powershell
az --version
```

Expected output:

```text
azure-cli                         2.xx.x
core                              2.xx.x
Python location: ....
```

---

# Step 2: Install Docker Desktop (Recommended)

Azure Container Apps deployments generally use container images.

Download:

[Docker Desktop Official Site](https://www.docker.com/products/docker-desktop/?utm_source=chatgpt.com)

Verify:

```powershell
docker --version
```

---

# Step 3: Install Visual Studio Code

Download:

[Visual Studio Code](https://code.visualstudio.com/?utm_source=chatgpt.com)

Install extensions:

* Azure Tools
* Docker
* Azure Container Apps
* Kubernetes
* YAML

---

# Step 4: Login to Azure

Open PowerShell or Command Prompt.

Login:

```powershell
az login
```

Browser window will open.

After login verify:

```powershell
az account show
```

Example:

```json
{
  "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "name": "Pay-As-You-Go",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

---

# Step 5: List Available Subscriptions

```powershell
az account list --output table
```

Example:

```text
Name             CloudName    SubscriptionId                 State
---------------  -----------  -----------------------------  -------
Dev Subscription AzureCloud   xxxxxxxx-xxxx-xxxx-xxxx       Enabled
```

---

# Step 6: Select Subscription

```powershell
az account set --subscription "Dev Subscription"
```

Verify:

```powershell
az account show --output table
```

---

# Step 7: Register Required Resource Providers

Azure Container Apps requires some providers.

Check registration:

```powershell
az provider list --query "[?namespace=='Microsoft.App']"
```

Register providers:

```powershell
az provider register --namespace Microsoft.App

az provider register --namespace Microsoft.OperationalInsights

az provider register --namespace Microsoft.ContainerRegistry
```

Check status:

```powershell
az provider show \
--namespace Microsoft.App \
--query registrationState
```

Expected:

```text
Registered
```

---

# Step 8: Install Container Apps Extension

Install extension:

```powershell
az extension add --name containerapp
```

Upgrade extension:

```powershell
az extension update --name containerapp
```

Verify:

```powershell
az containerapp --help
```

---

# Step 9: Install Azure Developer CLI (Optional)

Useful for samples.

```powershell
winget install Microsoft.Azd
```

Verify:

```powershell
azd version
```

---

# Step 10: Create Resource Group

Create resource group:

```powershell
az group create `
--name rg-containerapps-dev `
--location centralindia
```

Verify:

```powershell
az group list --output table
```

---

# Step 11: Create Log Analytics Workspace

Container Apps requires Log Analytics.

Create workspace:

```powershell
az monitor log-analytics workspace create `
--resource-group rg-containerapps-dev `
--workspace-name law-containerapps
```

Get Workspace ID:

```powershell
az monitor log-analytics workspace show `
-g rg-containerapps-dev `
-n law-containerapps `
--query customerId
```

Get Shared Key:

```powershell
az monitor log-analytics workspace get-shared-keys `
-g rg-containerapps-dev `
-n law-containerapps
```

Save these values.

---

# Step 12: Create Container App Environment

```powershell
az containerapp env create `
--name cae-dev `
--resource-group rg-containerapps-dev `
--location centralindia `
--logs-workspace-id <Workspace-ID> `
--logs-workspace-key <Workspace-Key>
```

Verify:

```powershell
az containerapp env list `
--resource-group rg-containerapps-dev `
--output table
```

---

# Step 13: Create First Container App

Deploy nginx:

```powershell
az containerapp create `
--name app-nginx `
--resource-group rg-containerapps-dev `
--environment cae-dev `
--image nginx `
--target-port 80 `
--ingress external
```

---

# Step 14: Get Application URL

```powershell
az containerapp show `
--name app-nginx `
--resource-group rg-containerapps-dev `
--query properties.configuration.ingress.fqdn
```

Open:

```text
https://<generated-url>
```

You should see:

```text
Welcome to nginx!
```

---

# Step 15: View Logs

Stream logs:

```powershell
az containerapp logs show `
--name app-nginx `
--resource-group rg-containerapps-dev `
--follow
```

---

# Step 16: Check Revisions

```powershell
az containerapp revision list `
--name app-nginx `
--resource-group rg-containerapps-dev `
--output table
```

---

# Step 17: Check Replicas

```powershell
az containerapp replica list `
--name app-nginx `
--resource-group rg-containerapps-dev `
--output table
```

---

# Step 18: Configure Autoscaling

```powershell
az containerapp update `
--name app-nginx `
--resource-group rg-containerapps-dev `
--min-replicas 0 `
--max-replicas 5
```

Check:

```powershell
az containerapp show `
--name app-nginx `
--resource-group rg-containerapps-dev `
--query properties.template.scale
```

---

# Step 19: Create Azure Container Registry (Optional)

```powershell
az acr create `
--name mycontaineracr123 `
--resource-group rg-containerapps-dev `
--sku Basic
```

Login:

```powershell
az acr login --name mycontaineracr123
```

---

# Step 20: Push Your Own Image

Tag image:

```powershell
docker tag myapp:latest `
mycontaineracr123.azurecr.io/myapp:v1
```

Push:

```powershell
docker push `
mycontaineracr123.azurecr.io/myapp:v1
```

Deploy:

```powershell
az containerapp update `
--name app-nginx `
--resource-group rg-containerapps-dev `
--image mycontaineracr123.azurecr.io/myapp:v1
```

---

# Useful Commands

### List Container Apps

```powershell
az containerapp list -o table
```

### Show Details

```powershell
az containerapp show `
-n app-nginx `
-g rg-containerapps-dev
```

### Delete App

```powershell
az containerapp delete `
-n app-nginx `
-g rg-containerapps-dev
```

### Delete Resource Group

```powershell
az group delete `
-n rg-containerapps-dev
```

---

# Complete Learning Flow

```text
Install Azure CLI
       ↓
Login
       ↓
Create Resource Group
       ↓
Create Log Analytics
       ↓
Create Container App Environment
       ↓
Deploy nginx
       ↓
Deploy Custom Images
       ↓
Configure Scaling
       ↓
Configure Revisions
       ↓
Configure Secrets
       ↓
Configure Managed Identity
       ↓
Integrate ACR
       ↓
Build CI/CD Pipeline
```

---

# Recommended Folder Structure for Labs

```text
AzureContainerAppsLabs
│
├── Lab01-CLI-Setup
├── Lab02-Environment
├── Lab03-nginx
├── Lab04-Scaling
├── Lab05-Revisions
├── Lab06-ACR
├── Lab07-Identity
├── Lab08-Dapr
└── Lab09-CICD
```

For learning purposes, I would suggest this sequence:

```text
Docker
   ↓
ACR
   ↓
Container Apps
   ↓
Jobs
   ↓
Dapr
   ↓
GitLab CI/CD
   ↓
AKS
```

This progression makes it much easier to understand the differences between serverless containers and full Kubernetes management.
