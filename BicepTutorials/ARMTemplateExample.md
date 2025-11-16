✔️ ARM templates **cannot** create a resource group at the resource-group scope—they must be deployed at the **subscription scope**.

---

# ✅ **1. ARM Template (create-rg.json)**

This template creates a resource group.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/subscriptionDeploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Resource Group to create"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "eastus",
      "metadata": {
        "description": "Location for the Resource Group"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Resources/resourceGroups",
      "apiVersion": "2021-04-01",
      "name": "[parameters('rgName')]",
      "location": "[parameters('location')]",
      "properties": {}
    }
  ]
}
```

---

# ✅ **2. Parameter File (optional: create-rg.parameters.json)**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "rgName": {
      "value": "my-demo-rg"
    },
    "location": {
      "value": "eastus"
    }
  }
}
```

---

# ✅ **3. Deploy the ARM Template using Azure CLI**

### **Using parameter file**

```bash
az deployment sub create \
  --name createRGDeployment \
  --location eastus \
  --template-file create-rg.json \
  --parameters @create-rg.parameters.json
```

### **Without a parameter file (pass inline)**

```bash
az deployment sub create \
  --name createRGDeployment \
  --location eastus \
  --template-file create-rg.json \
  --parameters rgName=my-demo-rg location=eastus
```

---

# 🎉 Output

This will create:

* A new **Resource Group** (example: `my-demo-rg`)
* In the selected **location** (example: `eastus`)

---

