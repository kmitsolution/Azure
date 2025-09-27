Great! Let’s dive into **Modules** — a key feature of Bicep for **reusable and organized templates**.

---

# 🔹 1. What is a Module?

A **module** in Bicep is a **separate Bicep file** that can be referenced from another Bicep file.
It allows you to:

* Break a large deployment into smaller, manageable pieces
* Reuse code across projects or environments
* Pass parameters and get outputs between templates

Think of it like a **function or class in programming** — it encapsulates logic.

---

# 🔹 2. Module Syntax

```bicep
module <symbolicName> '<path-to-module>.bicep' = {
  name: '<deployment-name>'
  params: {
    <param1>: <value1>
    <param2>: <value2>
  }
  dependsOn: [
    <resource-or-module-name> // optional
  ]
}
```

* `symbolicName` → internal reference name
* `name` → deployment name in Azure
* `params` → pass input parameters to the module
* `dependsOn` → optional, explicitly declare dependencies

---

# 🔹 3. Example: Reusable Storage Module

### Step 1: Create a module file `storageModule.bicep`

```bicep
param storageName string
param location string

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageId string = storageAccount.id
```

---

### Step 2: Call the module from `main.bicep`

```bicep
param baseName string = 'mydemo'
param location string = resourceGroup().location

var storageName = '${baseName}storage${uniqueString(resourceGroup().id)}'

module storage 'storageModule.bicep' = {
  name: 'deployStorage'
  params: {
    storageName: storageName
    location: location
  }
}

output storageAccountId string = storage.outputs.storageId
```

 Here:

* `storageModule.bicep` contains the storage account logic
* `main.bicep` calls the module and retrieves its output

---

# 🔹 4. Benefits of Modules

1. **Reusability** → Use the same module for Dev/Test/Prod
2. **Separation of Concerns** → Keep each template small and readable
3. **Outputs Linking** → Pass outputs from modules to the main template

---

# 🔹 5. Practice Exercise

1. Create a **storage module** (`storageModule.bicep`) as above.
2. Create `main.bicep` to call it.
3. Deploy:

```bash
az deployment group create \
  --resource-group MyDemoRG \
  --template-file main.bicep
```

4. Verify output:

```bash
az storage account show --name <your-storage-account-name> --resource-group MyDemoRG
```

 You’ve now **modularized your Bicep deployment** — a critical skill for real-world projects.

---

