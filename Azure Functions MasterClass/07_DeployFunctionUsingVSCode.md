Here’s a clear step-by-step guide for **creating and running an Azure Function locally using Visual Studio Code**:

---

### ✅ Step-by-Step: Create & Run Azure Function in VSCode

---

#### **1. Install Azure CLI**

* Download and install from:
  [https://learn.microsoft.com/en-us/cli/azure/install-azure-cli](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

---

#### **2. Install Azure Functions Extension in VSCode**

* Open **Visual Studio Code**
* Go to **Extensions** (`Ctrl+Shift+X`)
* Search and install: `Azure Functions`

---

#### **3. Sign In to Your Azure Account**

* In the **Azure panel** in VSCode, click **"Sign in to Azure..."**
* Complete authentication in the browser
* Select your **Azure subscription** after login

---

#### **4. Create New Azure Function Project**

* In the Azure panel, click **“Create Function...”**
* Provide:

  * **Language**: JavaScript / TypeScript / Python / C# etc.
  * **Location**
  * **Resource Group**
  * **Function App Name**
  * **Storage Account** (existing or new)
  * **Plan** (Consumption or Premium)

---

#### **5. Add a Trigger to the Project**

* After project is created, in VSCode:

  * Use `Command Palette` → `Azure Functions: Create Function`
  * Choose **HTTP trigger** (or any other)
  * Name it (e.g., `HttpTrigger1`)

---

#### **6. Run the Function Locally**

* Open the project folder in VSCode
* Click `Run > Start Debugging` or press `F5`
* The terminal will show something like:

```
Http Function running on http://localhost:7071/api/HttpTrigger1
```

---

#### **7. Connect to Storage Account (Optional)**

* During function creation, a storage account is linked.
* If needed, you can:

  * Browse the storage from Azure panel in VSCode
  * Connect existing accounts from Azure Explorer

---


