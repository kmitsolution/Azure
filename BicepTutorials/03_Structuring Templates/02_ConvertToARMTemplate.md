You can convert **Bicep → ARM Template (JSON)** using either the **Azure CLI** or **PowerShell**.
Both methods use the Bicep compiler built into Azure.

---

# ✅ **1. Convert using Azure CLI (Recommended)**

```bash
az bicep build --file main.bicep
```

This command generates:

```
main.json
```

Which is the ARM template equivalent of your Bicep file.

If your Bicep file is in another folder:

```bash
az bicep build --file ./infra/storage.bicep
```

---

# ✅ **2. Convert using Bicep CLI directly**

If you have the Bicep CLI installed:

```bash
bicep build main.bicep
```

This produces the ARM JSON template.

---

# ✅ **3. Convert using PowerShell**

```powershell
bicep build .\main.bicep
```

Same output — a `main.json` ARM template.

---

# 📌 **Example**

### **main.bicep**

```bicep
resource stg 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'mystorage123'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

### **Run conversion**

```bash
az bicep build --file main.bicep
```

### **Output: main.json**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-01-01",
      "name": "mystorage123",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2"
    }
  ]
}
```

---

# 📦 **4. Convert and See Template Without Saving (Preview Mode)**

```bash
az bicep build --file main.bicep --stdout
```

This prints the JSON to the terminal instead of creating a file.

---

# 🤝 Want me to convert your code?

If you share your **Bicep file**, I can convert it to ARM JSON for you right here.
