# Lesson 5: Groups in Microsoft Entra ID

This is one of the **most important topics** in Microsoft Entra ID. In real organizations, administrators **rarely assign permissions directly to users**. Instead, they assign permissions to **groups**, and then add users to those groups.

If you understand groups well, you'll understand how access is managed at scale.

---

# Learning Objectives

By the end of this lesson, you will be able to:

* Understand what a group is.
* Explain why groups are used.
* Differentiate Security Groups and Microsoft 365 Groups.
* Understand Assigned Membership and Dynamic Membership.
* Create and manage groups.
* Assign Azure RBAC roles to groups.
* Understand Group-Based Licensing.
* Apply groups in real-world enterprise scenarios.

---

# Why Do We Need Groups?

Imagine your company has **500 developers**.

All developers need access to:

* Azure DevOps
* Development Subscription
* Git Repository
* Azure Key Vault
* Storage Account

### Option 1: Assign permissions individually

```text
Developer1 → Storage
Developer2 → Storage
Developer3 → Storage
...
Developer500 → Storage
```

Problems:

* Time-consuming
* Easy to make mistakes
* Difficult to maintain
* Hard to audit

---

### Option 2: Use a Group

```text
Developers Group

├── Raman
├── Amit
├── Rahul
├── Priya
└── 496 more users
```

Assign permissions once:

```text
Developers Group

↓

Contributor Role

↓

Development Subscription
```

Every member automatically receives the required access.

**This is the recommended approach.**

---

# What is a Group?

A **Group** is a collection of users, devices, or service principals that are managed together.

Groups help administrators:

* Assign permissions
* Assign licenses
* Apply policies
* Simplify administration

---

# Types of Groups

Microsoft Entra ID supports two primary group types.

---

## 1. Security Group

A **Security Group** is used to control access.

Examples:

* Azure RBAC
* Storage access
* Virtual Machine access
* Key Vault access
* Enterprise Applications

Example:

```text
Developers

├── Raman
├── Rahul
├── Amit
```

Assign:

```text
Contributor

↓

Developers Group
```

All members inherit the Contributor role.

---

## 2. Microsoft 365 Group

Microsoft 365 Groups are designed for collaboration.

They automatically create shared resources such as:

* Outlook mailbox
* Shared calendar
* SharePoint site
* Microsoft Teams workspace
* Planner
* OneNote notebook

Example:

```text
Marketing Team
```

Members collaborate using Microsoft 365 services.

---

# Security Group vs Microsoft 365 Group

| Security Group      | Microsoft 365 Group    |
| ------------------- | ---------------------- |
| Access control      | Collaboration          |
| Azure RBAC          | Teams integration      |
| Key Vault access    | Outlook mailbox        |
| Storage permissions | SharePoint site        |
| VM permissions      | Planner & OneNote      |
| No shared mailbox   | Shared mailbox created |

---

# Membership Types

Microsoft Entra ID supports different ways to manage group membership.

---

## Assigned Membership

The administrator manually adds users.

Example:

```text
Developers

↓

Add Raman

Add Rahul

Add Priya
```

Best for:

* Small teams
* Stable memberships
* Administrative control

---

## Dynamic User Membership

Membership is determined automatically using rules.

Example Rule:

```text
Department = IT
```

Whenever a new user joins the IT department:

```text
Department

↓

IT

↓

Automatically Added

↓

Developers Group
```

No administrator action is required after the rule is created.

---

# Dynamic Membership Example

Suppose the company has:

| User  | Department |
| ----- | ---------- |
| Raman | IT         |
| Rahul | IT         |
| Amit  | HR         |
| Priya | Finance    |

Rule:

```text
Department = IT
```

Result:

```text
Developers

├── Raman
└── Rahul
```

If another employee's department changes to IT, they are automatically added to the group.

---

# Dynamic Device Groups

Dynamic rules can also be based on devices.

Example:

```text
Operating System = Windows
```

All Windows devices become members of that device group.

This is commonly used with Microsoft Intune.

---

# Creating a Group (Azure Portal)

Navigate to:

```text
Microsoft Entra ID

↓

Groups

↓

New Group
```

Choose:

Group Type:

* Security
* Microsoft 365

Enter:

* Group Name
* Description
* Membership Type
* Owners
* Members

Click **Create**.

---

# Example

Group Name

```text
Developers
```

Type

```text
Security Group
```

Membership

```text
Assigned
```

Members

```text
Raman

Rahul

Amit
```

---

# Group Owners

Every group should have one or more **Owners**.

Owners can:

* Add members
* Remove members
* Update group settings

Example:

```text
Developers

Owner

↓

Raman
```

Members do **not** automatically become owners.

---

# Group Members

Members inherit whatever permissions are assigned to the group.

Example:

```text
Developers

↓

Contributor Role

↓

Development Subscription
```

Every member receives Contributor permissions.

---

# Azure RBAC with Groups

Instead of assigning roles to individual users:

```text
Contributor

↓

Developers Group
```

Now:

```text
Raman

↓

Developers Group

↓

Contributor
```

This makes permission management much simpler.

---

# Group-Based Licensing

Suppose your organization has:

100 Microsoft 365 E5 licenses.

Instead of assigning licenses one by one:

Create:

```text
Developers
```

Assign:

```text
Microsoft 365 E5 License

↓

Developers Group
```

Every member automatically receives the license.

When someone leaves the group, the license is removed automatically.

---

# Enterprise Example

Company:

KMIT Solutions

Departments:

```text
HR

Finance

IT

Sales
```

Groups:

```text
HR Team

Finance Team

Developers

Azure Admins

Managers
```

Azure Roles:

```text
Azure Admins

↓

Owner

↓

Production Subscription
```

Developers:

```text
Developers

↓

Contributor

↓

Development Subscription
```

Finance:

```text
Finance Team

↓

Reader

↓

Cost Management
```

This design is scalable and easier to audit.

---

# Azure CLI

Create a group:

```bash
az ad group create \
    --display-name Developers \
    --mail-nickname Developers
```

List groups:

```bash
az ad group list --output table
```

Add a member:

```bash
az ad group member add \
    --group Developers \
    --member-id <UserObjectId>
```

---

# Microsoft Graph PowerShell

Create a group:

```powershell
New-MgGroup
```

Get groups:

```powershell
Get-MgGroup
```

Add a member:

```powershell
New-MgGroupMember
```

---

# Best Practices

✔ Assign permissions to **groups**, not individual users.

✔ Use meaningful names, such as:

```text
Azure-Developers

Azure-Admins

Finance-Readers

HR-Managers
```

✔ Assign at least one owner to every group.

✔ Use dynamic groups where possible to reduce manual administration.

✔ Use group-based licensing instead of assigning licenses individually.

---

# Common Mistakes

❌ Assigning Azure RBAC roles directly to users.

❌ Creating too many small groups with unclear purposes.

❌ Forgetting to assign a group owner.

❌ Using dynamic groups without understanding the rules.

❌ Naming groups inconsistently.

---

# Hands-on Lab

## Lab 1: Create Security Groups

Create:

```text
Developers

HR

Finance

AzureAdmins
```

Membership:

Assigned

---

## Lab 2: Add Members

Add users created in Lesson 4.

Verify the membership.

---

## Lab 3: Assign Azure RBAC

Assign:

```text
Developers

↓

Contributor

↓

Development Resource Group
```

Verify that members inherit the role.

---

## Lab 4: Create a Microsoft 365 Group

Create:

```text
Marketing Team
```

Observe the collaboration resources that are created (if Microsoft 365 services are available in your tenant).

---

## Lab 5: Dynamic Group (Optional)

Create a dynamic group with a rule such as:

```text
Department = IT
```

Create or update users so that some have `Department = IT`, then verify that membership updates automatically.

> **Note:** Dynamic group membership requires Microsoft Entra ID P1 or P2 licensing.

---

# Interview Questions

1. What is a Security Group?
2. What is a Microsoft 365 Group?
3. What is the difference between Assigned and Dynamic Membership?
4. Why should Azure RBAC roles be assigned to groups instead of users?
5. What is Group-Based Licensing?
6. What is the role of a Group Owner?
7. Can groups contain devices?
8. What license is required for Dynamic Groups?

---

# Practice Assignment

Create the following groups:

| Group Name       | Type          | Purpose                  |
| ---------------- | ------------- | ------------------------ |
| Azure-Developers | Security      | Development Azure access |
| Azure-Admins     | Security      | Administrative access    |
| Finance-Team     | Security      | Finance resources        |
| HR-Team          | Security      | HR resources             |
| Marketing-Team   | Microsoft 365 | Collaboration            |

Then:

1. Add users from Lesson 4.
2. Assign yourself as the owner of each group.
3. If you have the required license, create a dynamic group for all users in the IT department.
4. Assign a test RBAC role (for example, **Reader**) to one group at the resource group scope and verify that group members inherit the access.

---

Creating a **Dynamic User Group** in Microsoft Entra ID requires **Microsoft Entra ID P1 or P2** licensing. If your tenant only has the free edition, the **Dynamic membership** option will be unavailable.

---

# Step 1: Open Microsoft Entra Admin Center

1. Sign in to the Azure Portal.
2. Search for **Microsoft Entra ID**.
3. Go to **Groups**.
4. Click **All groups**.
5. Click **New group**.

---

# Step 2: Configure the Group

Fill in the details:

**Group Type**

```
Security
```

**Group Name**

```
IT-Developers
```

**Group Description**

```
All users in the IT department
```

---

# Step 3: Select Membership Type

Under **Membership type**, choose:

```
Dynamic User
```

You will now see the option:

```
Add dynamic query
```

Click it.

---

# Step 4: Build the Dynamic Rule

### Method 1: Rule Builder (Recommended)

Select:

| Field    | Value      |
| -------- | ---------- |
| Property | Department |
| Operator | Equals     |
| Value    | IT         |

The rule becomes:

```
Department Equals IT
```

Click **Save**.

---

### Method 2: Rule Syntax (Advanced)

Click **Edit** (or **Rule syntax**) and enter:

```text
(user.department -eq "IT")
```

Click **Save**.

---

# Step 5: Create the Group

Review the settings:

```
Group Type:
Security

Membership Type:
Dynamic User

Rule:
(user.department -eq "IT")
```

Click **Create**.

---

# Step 6: Verify Membership

Go to:

```
Groups

↓

IT-Developers

↓

Members
```

Initially, you may see:

```
Updating...
```

Membership evaluation can take a few minutes.

Once complete:

```
IT-Developers

├── Raman
├── Rahul
└── Amit
```

Only users whose **Department** is **IT** will appear.

---

# Step 7: Test the Dynamic Rule

Suppose you have:

| User  | Department |
| ----- | ---------- |
| Raman | IT         |
| Rahul | IT         |
| Priya | Finance    |
| Amit  | HR         |

Result:

```
IT-Developers

├── Raman
└── Rahul
```

Now edit **Amit**:

```
Department

↓

IT
```

After the dynamic group refreshes, Amit will be added automatically.

---

# Another Example: Job Title

Suppose you want all DevOps engineers.

Rule:

```text
(user.jobTitle -eq "DevOps Engineer")
```

Anyone with the job title **DevOps Engineer** becomes a member automatically.

---

# Multiple Conditions

Example:

Department = IT **AND** Country = India

Rule:

```text
(user.department -eq "IT") and
(user.country -eq "India")
```

Only users meeting both conditions are added.

---

# Using OR

Example:

Department is IT **or** HR

```text
(user.department -eq "IT") or
(user.department -eq "HR")
```

---

# Check the User Properties

If a user isn't added, verify the attribute values:

1. Go to **Microsoft Entra ID** → **Users**.
2. Select the user.
3. Open **Properties**.
4. Check fields like:

   * Department
   * Job Title
   * Country
   * City

The dynamic rule matches these properties exactly.

---

# Common Dynamic Rule Examples

| Requirement         | Rule                                                                |
| ------------------- | ------------------------------------------------------------------- |
| All IT users        | `(user.department -eq "IT")`                                        |
| All HR users        | `(user.department -eq "HR")`                                        |
| All Developers      | `(user.jobTitle -eq "Developer")`                                   |
| India employees     | `(user.country -eq "India")`                                        |
| Bangalore employees | `(user.city -eq "Bangalore")`                                       |
| Finance Managers    | `(user.department -eq "Finance") and (user.jobTitle -eq "Manager")` |

---

# Real-World Example

Suppose KMIT hires **50 new IT employees**.

Each new user has:

```
Department = IT
```

As soon as those users are created:

```
New Employee

↓

Department = IT

↓

Dynamic Rule Evaluated

↓

Automatically Added

↓

IT-Developers Group

↓

Gets Azure Permissions

↓

Gets Microsoft 365 License

↓

Gets Teams Access
```

No administrator needs to manually add them to the group.

---

## Important Notes

* Dynamic membership is supported only with **Microsoft Entra ID P1 or P2** licensing.
* Membership updates are **not always immediate**; they can take several minutes depending on the tenant and the number of objects being evaluated.
* Dynamic groups can be based on **user attributes** (Dynamic User) or **device attributes** (Dynamic Device), but you cannot mix users and devices in the same dynamic group.

If you're practicing in an Azure lab, I can also show you **how to create and test dynamic groups using Azure CLI and Microsoft Graph PowerShell**.


# What Comes Next?

In **Lesson 6: Administrative Units (AUs)**, you'll learn how large organizations delegate identity administration without giving global permissions.

We'll cover:

* What Administrative Units are.
* How they differ from Groups and Organizational Units (OUs).
* Delegated administration.
* Scoped administrative roles.
* Real-world scenarios such as separating administration by country, region, or business unit (for example, India, USA, and Europe).
