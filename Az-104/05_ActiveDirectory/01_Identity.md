--

# Lesson 1: Introduction to Identity and Microsoft Entra ID

## Learning Objectives

After this lesson, you will be able to:

* Explain what an identity is.
* Differentiate authentication from authorization.
* Understand what a directory service is.
* Understand the role of an Identity Provider (IdP).
* Explain why Microsoft Entra ID exists.
* Understand the difference between an Azure Account, Tenant, and Identity (high level).
* Navigate the Microsoft Entra Admin Center.

---

# Real World Analogy

Imagine a large company.

The company has:

* Employees
* Visitors
* Security guards
* Different office rooms

Suppose you arrive at the office.

The guard asks:

> "Who are you?"

You show your company ID card.

The guard verifies it.

If valid, you are allowed inside.

But you still cannot enter every room.

For example:

* HR Room
* Finance Room
* CEO Office

Each room has different permissions.

This simple example explains the two most important concepts in identity management:

**Authentication** → Proving who you are.

**Authorization** → Determining what you are allowed to access.

---

# What is an Identity?

An **identity** is simply **a unique representation of a user, application, service, or device**.

Examples include:

* A person
* A laptop
* A virtual machine
* An application
* A service
* A printer
* A mobile phone

Each identity has unique attributes such as:

* Username
* Email
* Employee ID
* Password (or another authentication method)
* Groups
* Roles

Example:

| Identity     | Username                                              |
| ------------ | ----------------------------------------------------- |
| Raman Sharma | [raman@company.com](mailto:raman@company.com)         |
| Employee A   | [employee1@company.com](mailto:employee1@company.com) |
| Azure VM     | vm-prod-01                                            |
| Web App      | payroll-app                                           |

In Azure, identities are not limited to people. Services and applications can also have identities.

---

# What is Authentication?

Authentication answers one question:

> **"Who are you?"**

The system verifies your identity.

Common authentication methods:

* Username + Password
* Fingerprint
* Face Recognition
* OTP
* Microsoft Authenticator
* Passkeys
* Smart Card
* FIDO2 Security Key

### Example

You enter:

```
Username:
raman@company.com

Password:
********
```

The system checks whether those credentials are correct.

If they are, you are authenticated.

Authentication does **not** decide what you can access.

---

# What is Authorization?

Authorization answers a different question:

> **"What are you allowed to do?"**

Once you've been authenticated, the system checks your permissions.

For example:

| User             | Permission      |
| ---------------- | --------------- |
| HR Employee      | HR Portal       |
| Finance Employee | Finance Portal  |
| CEO              | Everything      |
| Intern           | Training Portal |

Authorization is typically based on:

* Roles
* Groups
* Policies
* Permissions

---

# Authentication vs Authorization

| Authentication                        | Authorization                    |
| ------------------------------------- | -------------------------------- |
| Verifies identity                     | Determines permissions           |
| Happens first                         | Happens after authentication     |
| "Who are you?"                        | "What can you access?"           |
| Uses passwords, MFA, biometrics, etc. | Uses roles, groups, and policies |

Think of a hotel:

1. You show your passport at check-in (Authentication).
2. You receive a room key.
3. That key opens **only your room**, not every room (Authorization).

---

# What is a Directory?

A directory is a database that stores information about identities and related resources.

A simple directory might look like this:

| User  | Department | Role       |
| ----- | ---------- | ---------- |
| Raman | IT         | Admin      |
| Amit  | HR         | HR Manager |
| Priya | Finance    | Accountant |

A directory stores:

* Users
* Groups
* Devices
* Applications
* Roles
* Permissions

It is the central source for identity information.

---

# What is a Directory Service?

A **directory** stores identity data.

A **directory service** manages that data and provides services such as:

* User creation
* Password validation
* Group management
* Authentication
* Authorization
* Single Sign-On (SSO)

Examples:

* Active Directory Domain Services (on-premises)
* Microsoft Entra ID (cloud)
* Other LDAP-based directory services

---

# What is an Identity Provider (IdP)?

An Identity Provider is a system that authenticates users and issues proof of their identity to applications.

Instead of every application maintaining its own usernames and passwords, applications rely on a trusted Identity Provider.

### Without an Identity Provider

```
Email Application
      ↑
 Username & Password

HR Application
      ↑
 Username & Password

Payroll Application
      ↑
 Username & Password
```

You would manage separate credentials for every application.

### With an Identity Provider

```
             Microsoft Entra ID
                    │
        ┌───────────┼───────────┐
        │           │           │
    Outlook      HR App     Payroll App
```

You sign in once, and trusted applications use that authentication.

This is the foundation of **Single Sign-On (SSO)**.

---

# What is Microsoft Entra ID?

Microsoft Entra ID is Microsoft's **cloud-based Identity and Access Management (IAM)** service.

It manages identities for:

* Users
* Groups
* Applications
* Devices
* Managed Identities
* Service Principals

It provides:

* Authentication
* Authorization
* Single Sign-On
* Multi-Factor Authentication (MFA)
* Conditional Access
* Identity Protection
* Application Registration
* External Identity (B2B/B2C scenarios)

Whenever you sign in to Azure or many Microsoft cloud services, Microsoft Entra ID is involved.

---

# Why was Azure Active Directory renamed?

Microsoft renamed **Azure Active Directory (Azure AD)** to **Microsoft Entra ID** to make its purpose clearer.

Reasons include:

* It is not a direct replacement for traditional Windows Active Directory.
* It is a cloud-native identity platform.
* It is part of the broader Microsoft Entra family of identity products.

Only the name changed. Existing features and concepts largely remained the same.

---

# Where is Microsoft Entra ID Used?

Many Microsoft services depend on it, including:

* Azure Portal
* Microsoft 365
* Azure Virtual Machines
* Azure Kubernetes Service (AKS)
* Azure Storage
* Azure SQL Database
* Azure Key Vault
* Azure DevOps
* Custom web and mobile applications

---

# High-Level View

```
            User
              │
              ▼
     Microsoft Entra ID
              │
   Authentication & Authorization
              │
      Azure Resources & Apps
```

---

# Hands-on Lab 1: Explore Microsoft Entra ID

**Prerequisites:** An Azure subscription (Free or Paid).

### Step 1

Sign in to the Azure Portal.

### Step 2

Search for:

```
Microsoft Entra ID
```

or

```
Entra ID
```

### Step 3

Open the **Overview** page and identify:

* Tenant name
* Tenant ID
* Primary domain
* License information

Don't worry if these terms are unfamiliar—we'll cover them in the next lessons.

### Step 4

Explore the left navigation menu and note these sections:

* Overview
* Users
* Groups
* Enterprise Applications
* App Registrations
* Devices
* Roles and Administrators
* Authentication Methods
* Identity Protection (if your license includes it)
* Monitoring

Do not make changes yet; just become familiar with the interface.

---

# Key Takeaways

* **Identity** represents a user, application, service, or device.
* **Authentication** verifies who you are.
* **Authorization** determines what you can do.
* A **directory** stores identity information.
* A **directory service** manages identities and authentication.
* An **Identity Provider (IdP)** authenticates users for applications.
* **Microsoft Entra ID** is Microsoft's cloud-based Identity and Access Management (IAM) platform and is central to Azure security.

---

# Interview Questions

1. What is an identity?
2. Explain the difference between authentication and authorization with an example.
3. What is an Identity Provider (IdP)?
4. What is a directory service?
5. What is Microsoft Entra ID?
6. Why was Azure Active Directory renamed to Microsoft Entra ID?
7. Give examples of identities other than users.
8. How does Single Sign-On (SSO) improve the user experience?

---

# Practice Assignment

1. Define the following in your own words:

   * Identity
   * Authentication
   * Authorization
   * Directory
   * Directory Service
   * Identity Provider

2. Log in to the Azure Portal and locate your:

   * Tenant Name
   * Tenant ID
   * Primary Domain

3. Explore the Microsoft Entra ID menu and write down the purpose of each of these sections:

   * Users
   * Groups
   * App registrations
   * Enterprise applications
   * Roles and administrators

Once you've completed the assignment or if you have questions, we'll move on to **Lesson 2: Active Directory Fundamentals**, where you'll learn domains, forests, organizational units (OUs), domain controllers, LDAP, Kerberos, and how on-premises Active Directory compares to Microsoft Entra ID.
