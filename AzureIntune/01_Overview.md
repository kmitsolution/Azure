



Since you're already learning **Microsoft Entra ID**, this is the perfect time to learn **Microsoft Intune** because the two products work together extensively.

# What is Microsoft Intune?

**Microsoft Intune** is Microsoft's **cloud-based Endpoint Management (EMM/MDM/MAM)** solution.

It allows an organization to **manage, secure, configure, monitor, and update** devices from anywhere.

Think of Intune as an **IT administrator for all company devices**.

It manages:

- Windows laptops/desktops
- macOS
- Android phones
- iPhones/iPads
- Microsoft Teams Rooms
- Virtual endpoints

---

# Simple Analogy

Imagine a company has **1000 employees**.

Without Intune:

- Every laptop must be configured manually.
- Software must be installed manually.
- Password policies are manual.
- Antivirus updates are manual.
- If an employee leaves, data remains on the laptop.

With Intune:

Administrator simply creates policies.

Whenever a new device joins the company:

- Security settings apply automatically.
- Office installs automatically.
- VPN config applies automatically.
- Wi-Fi config applies automatically.
- BitLocker enables automatically.
- Antivirus settings apply automatically.
- Company apps install automatically.

No manual work.

---

# What problems does Intune solve?

Without Intune:

❌ Every computer is different

❌ Users install anything

❌ Passwords are weak

❌ Devices aren't encrypted

❌ Lost laptop exposes company data

❌ Difficult to manage remote employees

---

With Intune:

✔ Centralized management

✔ Device security

✔ App management

✔ Remote management

✔ Compliance monitoring

✔ Remote wipe

✔ Automatic deployment

---

# Intune Architecture

```
                    Microsoft Entra ID
                           │
                           │
             Authentication & Identity
                           │
                           ▼
                    Microsoft Intune
                           │
        ┌──────────────┬───────────────┐
        │              │               │
        ▼              ▼               ▼
 Windows Devices   Android Phones   iPhones
        │
        ▼
 Policies
 Apps
 Security
 Updates
 Compliance
```

---

# Components of Intune

## 1. Device Management (MDM)

Manage the complete device.

Example:

- Laptop
- Desktop
- Mobile
- Tablet

Administrator can

- Lock device
- Restart
- Reset
- Rename
- Configure
- Encrypt
- Wipe

---

## 2. Mobile Application Management (MAM)

Manages only applications.

Example:

Company allows employees to use personal phones.

Company only controls:

- Outlook
- Teams
- OneDrive

Personal photos remain untouched.

This is called **BYOD (Bring Your Own Device)**.

---

## 3. Compliance Policies

Checks whether device follows company rules.

Example Policy:

```
Password Required

BitLocker Enabled

Firewall ON

Antivirus Running

Windows Updated

OS Version >= Windows 11

TPM Enabled
```

If all conditions satisfy

Device = Compliant

Otherwise

Device = Non-Compliant

---

# 4. Configuration Profiles

Automatically configure devices.

Example

```
Wallpaper

Desktop Shortcuts

VPN

Wi-Fi

Printer

Edge Settings

USB Restrictions

Lock Screen
```

---

# 5. Application Deployment

Install applications remotely.

Example

```
Microsoft Office

Chrome

Adobe Reader

Visual Studio Code

Teams

Zoom

Company ERP
```

No manual installation.

---

# 6. Windows Update Management

Administrator decides

- When updates install
- Restart timing
- Feature Updates
- Security Updates

---

# 7. Endpoint Security

Configure

- Microsoft Defender
- Firewall
- BitLocker
- Attack Surface Reduction
- Antivirus

---

# 8. Remote Actions

Administrator can

- Restart
- Sync
- Wipe
- Fresh Start
- Retire
- Lock

Even if user is working from another country.

---

# Device Lifecycle

```
New Laptop Purchased
          │
          ▼
Windows Installed
          │
          ▼
Join Microsoft Entra ID
          │
          ▼
Enroll into Intune
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
User Starts Working
```

---

# How Intune works with Entra ID

Suppose a new employee joins.

### Step 1

Create user in Entra ID.

```
Raman
```

↓

### Step 2

Assign Microsoft Intune License.

↓

### Step 3

Employee signs into Windows.

↓

### Step 4

Device automatically joins Entra ID.

↓

### Step 5

Device enrolls into Intune.

↓

### Step 6

Policies apply.

↓

### Step 7

Applications install.

↓

Employee starts working.

---

# Real Company Example

Company policy:

- Minimum password length = 12
- BitLocker enabled
- Office installed
- Teams installed
- Outlook configured
- VPN configured
- USB disabled
- Firewall enabled
- Defender enabled

Employee buys a new laptop.

He simply signs in using his company account.

Within about 15–30 minutes:

✓ Laptop is configured

✓ Office installed

✓ Teams installed

✓ Outlook configured

✓ VPN configured

✓ Security policies applied

No IT engineer needs to physically touch the device.

---

# Intune Licensing

Intune is included with several Microsoft subscriptions, such as:

- Microsoft Intune Plan 1
- Microsoft 365 Business Premium
- Microsoft 365 E3
- Microsoft 365 E5
- Enterprise Mobility + Security (EMS) E3/E5

The exact features available depend on the license assigned.

---

# Where to manage Intune

Administrators use the **Microsoft Intune admin center** to manage devices, apps, policies, and compliance.

---

# Intune vs Entra ID

| Microsoft Entra ID | Microsoft Intune |
|--------------------|------------------|
| Identity management | Device management |
| Users | Devices |
| Authentication | Configuration |
| Groups | Compliance |
| MFA | App deployment |
| Conditional Access | Security policies |
| Single Sign-On | Remote management |

**Simple way to remember:**

- **Microsoft Entra ID** answers: **"Who are you?"**
- **Microsoft Intune** answers: **"Is your device trusted, configured, and secure?"**

---

# AZ-104 Perspective

For the AZ-104 exam, you should understand:

- What Microsoft Intune is
- Device enrollment methods
- Entra ID join vs Microsoft Entra hybrid join
- Compliance policies
- Configuration profiles
- App deployment
- Windows Autopilot (zero-touch device provisioning)
- Integration with Conditional Access
- Endpoint security basics

## Learning Path Recommendation

We'll cover Intune step by step, just as we did for Entra ID:

1. **Lesson 1:** Intune Architecture and Components ✅ (today)
2. **Lesson 2:** Device Enrollment (Windows, Android, iOS)
3. **Lesson 3:** Microsoft Entra Join vs Hybrid Join vs Registered
4. **Lesson 4:** Configuration Profiles
5. **Lesson 5:** Compliance Policies
6. **Lesson 6:** Application Deployment
7. **Lesson 7:** Endpoint Security
8. **Lesson 8:** Windows Autopilot
9. **Lesson 9:** Conditional Access with Intune
10. **Lesson 10:** Complete hands-on lab from scratch using the Azure portal

This sequence builds from the basics to real-world administration and aligns well with the skills needed for both AZ-104 and day-to-day Azure administration.
