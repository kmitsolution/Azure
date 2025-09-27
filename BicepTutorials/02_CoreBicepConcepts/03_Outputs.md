# 🔹 1. What are Outputs?

**Outputs** are values that a Bicep template returns after deployment.
They’re useful for:

* Showing resource IDs, endpoints, or connection strings
* Passing values to another template or module
* Verifying deployment results

---

# 🔹 2. Output Syntax

```bicep
output <name> <type> = <expression>
```

* `name` → name of the output
* `type` → data type (`string`, `int`, `bool`, `array`, `object`)
* `expression` → the value you want to output

---

# 🔹 3. Example: Output Storage Account ID

```bicep
param storageName string = 'mybicepstorage${uniqueString(resourceGroup().id)}'
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageId string = storageAccount.id
output storageNameOut string = storageAccount.name
```

**Explanation:**

* `storageId` → the unique Azure resource ID
* `storageNameOut` → the actual name of the storage account deployed

---

# 🔹 4. Deploy and See Outputs

```bash
az deployment group create \
  --resource-group MyDemoRG \
  --template-file outputsDemo.bicep
```

After deployment, the CLI will display something like:

```json
{
  "storageId": "/subscriptions/<sub-id>/resourceGroups/MyDemoRG/providers/Microsoft.Storage/storageAccounts/mybicepstorage1234",
  "storageNameOut": "mybicepstorage1234"
}
```

---

# 🔹 5. Best Practices

1. Only output what’s **necessary** (e.g., IDs, endpoints, secrets should come from Key Vault).
2. Use outputs for **linking resources across modules**.
3. Match the output type with the actual data (`string`, `array`, `object`).

---

# 🔹 6. Practice Exercise

1. Create `outputsDemo.bicep`:

```bicep
param baseName string = 'mydemo'
param location string = resourceGroup().location

var storageName = '${baseName}storage${uniqueString(resourceGroup().id)}'

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageId string = storageAccount.id
output storageNameOut string = storageAccount.name
```

2. Deploy:

```bash
az deployment group create \
  --resource-group MyDemoRG \
  --template-file outputsDemo.bicep
```

 You now see **storage account ID and name** as outputs.

---

Next, we can move to **Modules**, which let you **reuse templates** and organize large deployments.


