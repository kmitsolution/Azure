Let’s write your **first Bicep template** — a simple “Hello Azure” that deploys a **storage account**. This will help you see how Bicep works in practice.

---

# 🔹 Step 1: Create a Bicep file

1. Open **VS Code**
2. Create a new file: `helloStorage.bicep`

---

# 🔹 Step 2: Write the Bicep code

```bicep
// Parameters (values you can change when deploying)
param storageName string = 'hellostorage${uniqueString(resourceGroup().id)}'
param location string = resourceGroup().location

// Resource declaration
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

// Output the resource ID
output storageId string = storageAccount.id
```

**Explanation:**

* `param` → allows you to pass values dynamically.
* `resource` → defines the Azure resource (storage account in this case).
* `output` → returns the storage account ID after deployment.
* `uniqueString(resourceGroup().id)` → ensures the storage account name is unique (required by Azure).

---

# 🔹 Step 3: Create a Resource Group

```bash
az group create --name MyDemoRG --location eastus
```

---

# 🔹 Step 4: Deploy the Bicep Template

```bash
az deployment group create \
  --resource-group MyDemoRG \
  --template-file helloStorage.bicep
```

✅ After deployment, the output will show the **resource ID** of your storage account.

---

# 🔹 Step 5: Verify Deployment

```bash
az storage account show --name <your-storage-account-name> --resource-group MyDemoRG
```

You should see your storage account details.

---

This is your **“Hello Azure”** moment — you’ve written and deployed a real Bicep template! 🎉

---

Next, we can **add parameters, variables, loops, and modules** to make your Bicep templates more flexible and reusable.

Do you want to move to **Core Bicep Concepts** next?
