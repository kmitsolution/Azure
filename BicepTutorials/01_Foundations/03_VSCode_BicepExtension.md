Let’s set up **Visual Studio Code** with the **Bicep extension** so you can write and edit Bicep files efficiently.

---

# 🔹 Step 1: Install Visual Studio Code

###  Windows & macOS

1. Go to the official download page: [VS Code Download](https://code.visualstudio.com/download)
2. Download the installer for your OS.
3. Follow the installation steps (default options are fine).

###  Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install software-properties-common apt-transport-https wget
wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
sudo apt update
sudo apt install code
```

 Verify installation:

```bash
code --version
```

---

# 🔹 Step 2: Install the Bicep Extension

1. Open **VS Code**
2. Go to the **Extensions** panel (Ctrl+Shift+X or Cmd+Shift+X)
3. Search for: `Bicep`
4. Install **“Bicep” by Microsoft**

**Features you get:**

* Syntax highlighting
* IntelliSense (autocomplete) for Azure resources
* Error checking while you type
* Snippets for common resource templates

---

# 🔹 Step 3: Verify the Extension

1. Create a new file with extension `.bicep`
2. Type a small snippet, for example:

```bicep
param storageName string
```

3. You should see autocomplete and syntax highlighting →  confirms Bicep extension is working.

---

At this point:

* VS Code installed
* Bicep extension installed
* Ready to start writing Bicep templates

---


