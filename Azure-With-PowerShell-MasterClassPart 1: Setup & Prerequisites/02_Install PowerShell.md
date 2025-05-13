## PowerShell Installation 

---

## 🔍 Step 1: **Check Installed PowerShell Version**

Run this in **any PowerShell window**:

```powershell
$PSVersionTable
```

Look for `PSVersion`. For example:

```text
Name                           Value
----                           -----
PSVersion                      5.1.19041.3636
```

* If it's **5.1**, you're using **Windows PowerShell** (comes with Windows).
* If it's **7.x or higher**, you're using **PowerShell (Core)**, the newer cross-platform version.

---

## 💡 Windows PowerShell (Blue Screen) vs PowerShell (Black Screen)

| Feature              | Windows PowerShell             | PowerShell (Core / 7+)            |
| -------------------- | ------------------------------ | --------------------------------- |
| UI Color             | Blue console window            | Black console window (by default) |
| Version Range        | 5.1 and below                  | 6.x, 7.x and above                |
| Based On             | .NET Framework                 | .NET Core / .NET 5+               |
| Cross-platform       | ❌ Windows-only                 | ✅ Windows, macOS, Linux           |
| Actively Updated     | ❌ (Legacy)                     | ✅ (Actively developed)            |
| Module Compatibility | Supports older Windows modules | May need compatibility mode       |

So:

* **Windows PowerShell** = Legacy, built into Windows (blue screen)
* **PowerShell (Core)** = New, modern version you install manually (black screen)

---

## ✅ Step 2: **Install the Latest PowerShell (PowerShell 7+)**

You can install PowerShell 7+ **alongside** Windows PowerShell. They don't conflict.

### 📥 Option 1: Install via MSI (Windows Installer)

1. Go to: [https://github.com/PowerShell/PowerShell/releases](https://github.com/PowerShell/PowerShell/releases)
2. Download the latest `.msi` installer for your system (e.g., `PowerShell-7.4.1-win-x64.msi`)
3. Run the installer
4. Launch "PowerShell 7" from Start Menu or Terminal
Perfect — here's the updated tutorial with **Option 2** added to **Step 2** using the official Microsoft documentation link and all available installation methods.

---


### 🌐 **Option 2: Use Official Microsoft Docs – Multiple Installation Methods**

Microsoft provides a full guide with various installation options, including:

* ✅ **WinGet (recommended if using Windows 10/11)**

  ```powershell
  winget install --id Microsoft.Powershell --source winget
  ```

* ✅ **MSI installer** (GUI-based, good for most users)

* ✅ **ZIP package** (portable use)

* ✅ **.NET Global Tool** (developer-friendly)

* ✅ **Microsoft Store** (auto updates, no admin needed)

* ✅ **Preview versions** (for testing new features)

* ✅ **Upgrading existing installations**

* ✅ **Installing on special platforms**:

  * Windows 10 IoT Enterprise
  * Windows 10 IoT Core
  * Nano Server

🔗 Full guide:
👉 [Installing PowerShell on Windows (Microsoft Docs)](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5)

Choose the installation method that best suits your setup (admin access, update preference, environment, etc.)

---



---

### ✅ Step 3: **Verify New Version Installed**

Open the new PowerShell window and run:

```powershell
$PSVersionTable
```

You should see something like:

```text
PSVersion                      7.4.1
```

---

## 🤔 Can You Have Multiple Versions Installed?

Yes! ✅

You can have:

* **Windows PowerShell 5.1** (always there on Windows)
* **PowerShell 7.x** (installed separately)

They **run side by side**. You can even call one from the other if needed.

---


