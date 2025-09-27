Let’s explore **Bicep Playground**, a browser-based tool to practice Bicep without installing anything. It’s great for learning and experimenting.

---

# 🔹 What is Bicep Playground?

* A **web-based editor** for writing Bicep code.
* Lets you **see the compiled ARM JSON** in real-time.
* No need to install VS Code, Azure CLI, or Bicep CLI.
* Great for testing and learning Bicep syntax.

Official link: [Bicep Playground](https://bicepdemo.z22.web.core.windows.net/)

---

# 🔹 Key Features

1. **Code Editor**

   * Write Bicep code in the left pane.
   * Syntax highlighting, IntelliSense, and snippets are available.

2. **ARM Template Viewer**

   * Right pane shows the **compiled ARM JSON** equivalent.
   * Helps you understand what Bicep is doing behind the scenes.

3. **Sample Templates**

   * Click **Examples** → see predefined templates (storage, VMs, networks).
   * Modify them to learn faster.

4. **Copy & Paste**

   * Copy Bicep code to VS Code for real deployment later.

---

# 🔹 Practice Exercise in Playground

1. Open [Bicep Playground](https://bicepdemo.z22.web.core.windows.net/)
2. Create a small template:

```bicep
param storageName string = 'mybicepstorage'
param location string = 'eastus'

resource storage 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

3. Watch the **right pane** → it will show the equivalent ARM JSON.
4. Experiment: change `sku` or `kind`, and see how the ARM JSON updates.

---

 **Benefits:**

* No setup required → start coding immediately
* See Bicep → ARM mapping
* Learn resource syntax safely

---

Next, we can move on to your **first Bicep deployment** in a real Azure subscription, using VS Code and Azure CLI.

Do you want to do that next?
