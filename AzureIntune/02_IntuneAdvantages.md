# Lesson 1: Intune Architecture and Setup

## Learning Objectives

By the end of this lesson, you will understand:

* What Intune is
* Why organizations use it
* How Intune works
* The major Intune components
* How Intune integrates with Microsoft Entra ID
* What happens when a device is enrolled

---

# Before Intune

Imagine a company with 500 employees.

Each employee gets:

* A laptop
* Microsoft Office
* Outlook
* Teams
* VPN
* Antivirus
* Company security settings

Without Intune, the IT team must configure every laptop manually.

For 500 laptops, this takes a huge amount of time.

---

# After Intune

With Intune, the IT administrator creates policies **once**.

When a user signs in to a new company laptop:

* Office installs automatically.
* Teams installs automatically.
* Outlook configures automatically.
* VPN settings are applied.
* BitLocker is enabled.
* Security policies are enforced.
* Company Wi-Fi is configured.

No manual configuration is needed.

---

# What Intune Does

Think of Intune as a cloud-based device management service.

It manages:

```
          Devices
              │
      ┌───────┼────────┐
      │       │        │
 Windows  Android   iPhone
      │       │        │
      └───────┼────────┘
              │
        Microsoft Intune
              │
      Policies & Security
```

---

# Intune Components

## 1. Device Enrollment

Before Intune can manage a device, the device must be enrolled.

Example:

```
Windows Laptop
      │
      ▼
Enroll into Intune
      │
      ▼
Managed Device
```

Without enrollment:

❌ Intune cannot control the device.

---

## 2. Configuration Profiles

These configure devices automatically.

Example settings:

* Wi-Fi
* VPN
* Wallpaper
* Desktop shortcuts
* Edge browser settings
* Lock screen
* USB restrictions

Example:

Every company laptop automatically connects to the corporate Wi-Fi without the user entering the password.

---

## 3. Compliance Policies

These verify whether devices meet company security requirements.

Example policy:

```
Password Required

Firewall Enabled

BitLocker Enabled

Windows Updated

TPM Enabled
```

If every rule passes:

```
Compliant
```

Otherwise:

```
Non-Compliant
```

---

## 4. Application Deployment

Instead of asking users to install software manually, Intune installs applications automatically.

Examples:

* Microsoft Office
* Microsoft Teams
* Google Chrome
* Visual Studio Code
* Adobe Reader
* Company ERP applications

---

## 5. Endpoint Security

Protects devices by configuring security settings such as:

* Microsoft Defender
* BitLocker
* Firewall
* Antivirus
* Attack Surface Reduction (ASR)

---

## 6. Remote Actions

If a laptop is lost or stolen, the administrator can:

* Restart
* Sync
* Lock
* Retire
* Wipe
* Fresh Start

These actions can be performed remotely.

---

# How Intune Works with Entra ID

```
Create User
      │
      ▼
Microsoft Entra ID
      │
      ▼
Assign Intune License
      │
      ▼
User Signs into Device
      │
      ▼
Device Joins Entra ID
      │
      ▼
Enrolls into Intune
      │
      ▼
Policies Download
      │
      ▼
Apps Install
      │
      ▼
Compliance Check
      │
      ▼
User Ready to Work
```

---

# Real-Life Example

A new employee named **Rahul** joins the company.

The administrator:

* Creates Rahul's account in Microsoft Entra ID.
* Assigns a Microsoft Intune license.
* Ships a new laptop to Rahul.

Rahul turns on the laptop and signs in with his company account.

Within a short time:

* Office is installed.
* Teams is installed.
* Outlook is configured.
* Company Wi-Fi and VPN settings are applied.
* BitLocker is enabled.
* Firewall and Defender are configured.
* Required company apps are installed.

Rahul can start working without needing IT to configure the laptop manually.

---

# Intune and Entra ID: Responsibilities

| Microsoft Entra ID   | Microsoft Intune     |
| -------------------- | -------------------- |
| User identities      | Device management    |
| Authentication       | Device configuration |
| Groups               | Compliance policies  |
| Single Sign-On (SSO) | App deployment       |
| Conditional Access   | Endpoint security    |

A simple way to remember:

* **Microsoft Entra ID** verifies **who the user is**.
* **Microsoft Intune** manages **how the user's device is configured and secured**.

---

# Hands-on Lab 1: Explore the Intune Admin Center

### Prerequisites

* An Azure subscription.
* A Microsoft 365 tenant with an Intune license (trial or paid).

### Steps

1. Sign in to the Azure portal.
2. Search for **Microsoft Intune**.
3. Open the **Microsoft Intune admin center**.
4. Explore these sections (don't make changes yet):

   * **Dashboard**
   * **Devices**
   * **Apps**
   * **Users**
   * **Groups**
   * **Endpoint Security**
   * **Reports**
   * **Tenant Administration**

Your goal is simply to become familiar with the interface.

---

# Lesson 1 Summary

You should now understand:

* What Microsoft Intune is.
* Why organizations use it.
* The main components of Intune.
* How Intune integrates with Microsoft Entra ID.
* The lifecycle of an enrolled device.
* Where to manage Intune.

---

# Knowledge Check

Try answering these questions:

1. What is the primary purpose of Microsoft Intune?
2. What is the difference between a **Configuration Profile** and a **Compliance Policy**?
3. Can Intune manage a device that has **not** been enrolled?
4. Which service is responsible for **user identity**: Microsoft Entra ID or Intune?
5. Which Intune feature lets you remotely erase data from a lost laptop?

Once you're comfortable with these concepts, we'll move to **Lesson 2: Device Enrollment**, where you'll learn how Windows, Android, and iOS devices are enrolled into Intune and the differences between enrollment methods.
