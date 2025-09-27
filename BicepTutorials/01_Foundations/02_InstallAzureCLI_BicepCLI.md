Let’s install both **Azure CLI** and **Bicep CLI** step by step.

---

# 🔹 Step 1: Install Azure CLI

Azure CLI is the command-line tool you’ll use to interact with Azure and deploy Bicep templates.

###  Windows

Run in **PowerShell**:

```powershell
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
```

###  macOS

Run in **Terminal** (requires [Homebrew](https://brew.sh/)):

```bash
brew update && brew install azure-cli
```

###  Linux (Ubuntu/Debian)

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

 Verify installation:

```bash
az --version
```

---

# 🔹 Step 2: Install Bicep CLI

Bicep is included as part of Azure CLI, so installation is simple.

###  Install via Azure CLI

```bash
az bicep install
```

###  Upgrade later (when new versions release)

```bash
az bicep upgrade
```

 Verify installation:

```bash
az bicep version
```

---

# 🔹 Step 3: Login to Azure

Make sure you can connect to your Azure subscription:

```bash
az login
```

A browser window will open → log in with your Azure account.

---

 At this point, you have:

* **Azure CLI** installed
* **Bicep CLI** installed and linked into Azure CLI
* Ability to deploy resources with `az deployment`

---

Do you want me to also give you a **single copy-paste script** (for Windows/Mac/Linux) that installs **both CLI tools in one go**?
