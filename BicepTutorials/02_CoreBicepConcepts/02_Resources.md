# 🔹 1. What is a Resource?

In Bicep, a **resource** represents an **Azure service or object** you want to deploy.
Examples: Storage Account, Virtual Network, App Service, SQL Database.

Every resource declaration defines:

* **Type** → Azure resource type (e.g., `Microsoft.Storage/storageAccounts`)
* **API Version** → which version of the Azure REST API to use
* **Name** → the resource name
* **Location** → region where the resource is deployed
* **Properties** → configuration for the resource

---

# 🔹 2. Resource Syntax

```bicep
resource <symbolicName> '<resourceType>@<apiVersion>' = {
  name: <resourceName>
  location: <location>
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}
```

* `symbolicName` → internal name used in Bicep to reference the resource
* `<resourceType>` → the type of Azure resource
* `<apiVersion>` → the API version to use for deployment

---

# 🔹 3. Simple Example

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
  properties: {
    accessTier: 'Hot'
  }
}

output storageId string = storageAccount.id
```

 This deploys a **Storage Account** with:

* Standard locally-redundant storage
* StorageV2 kind
* Hot access tier

---

# 🔹 4. Key Points About Resources

1. **Dependencies**

   * Resources are deployed in the right order based on references.
   * Example: If App Service depends on Storage Account, Bicep figures out the order automatically.

2. **Existing Resources**

   * You can reference a resource that already exists:

   ```bicep
   resource existingStorage 'Microsoft.Storage/storageAccounts@2023-01-01' existing = {
     name: 'mystorageaccount'
   }
   ```

3. **Loops & Conditions** (advanced)

   * You can deploy multiple resources dynamically or conditionally (covered later).

---

# 🔹 5. Practice Exercise

1. Create `resourcesDemo.bicep`:

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
  properties: {
    accessTier: 'Hot'
  }
}

output storageId string = storageAccount.id
```

2. Deploy it:

```bash
az deployment group create \
  --resource-group MyDemoRG \
  --template-file resourcesDemo.bicep
```

 You now have a **fully deployed Storage Account**, and you can reference its `id` for other resources.

---

Next, we can cover **Outputs** — how to return values from your Bicep template, which is very useful for chaining resources.


