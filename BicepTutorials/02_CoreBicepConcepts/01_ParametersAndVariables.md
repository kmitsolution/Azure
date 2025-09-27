Perfect! Let’s dive into **Core Bicep Concepts: Parameters and Variables**. These are fundamental for making your templates **flexible, reusable, and dynamic**.

---

# 🔹 1. Parameters

**Parameters** are inputs you pass to a Bicep template at deployment time.
They allow you to **customize resources without changing the code**.

### Syntax

```bicep
param <name> <type> [= <defaultValue>]
```

* `name` → the variable name
* `type` → data type (`string`, `int`, `bool`, `array`, `object`)
* `defaultValue` → optional default

### Example

```bicep
param storageName string = 'mystorage${uniqueString(resourceGroup().id)}'
param location string = 'eastus'
param enableHttps bool = true
```

You can **override parameters** at deployment:

```bash
az deployment group create \
  --resource-group MyDemoRG \
  --template-file mytemplate.bicep \
  --parameters storageName=customstorage location=westus
```

 **Benefit:** Same template works for multiple environments (Dev/Test/Prod).

---

# 🔹 2. Variables

**Variables** store **calculated or reused values** inside the template.
They are evaluated at deployment and **cannot be overridden** like parameters.

### Syntax

```bicep
var <name> = <expression>
```

### Example

```bicep
param baseName string = 'myapp'

var storageName = '${baseName}storage${uniqueString(resourceGroup().id)}'
var appServiceName = '${baseName}-webapp'
```

You can now use these variables in resource definitions:

```bicep
resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

 **Benefit:** Avoid repeating the same value multiple times and make template cleaner.

---

# 🔹 3. Quick Comparison

| Feature     | Parameter                      | Variable                   |
| ----------- | ------------------------------ | -------------------------- |
| Overridable | Yes, at deployment             | No, fixed in template      |
| Scope       | Input to template              | Computed inside template   |
| Use case    | Environment values, user input | Derived or reusable values |

---

# 🔹 4. Practice Exercise

1. Create a Bicep file `paramVarDemo.bicep`:

```bicep
param baseName string = 'mydemo'
param location string = resourceGroup().location

var storageName = '${baseName}storage${uniqueString(resourceGroup().id)}'
var appServiceName = '${baseName}-webapp'

resource storage 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageId string = storage.id
output appName string = appServiceName
```

2. Deploy it:

```bash
az deployment group create \
  --resource-group MyDemoRG \
  --template-file paramVarDemo.bicep \
  --parameters baseName=customapp
```

 Output will show your **storage account ID** and the **computed app name**.

---


