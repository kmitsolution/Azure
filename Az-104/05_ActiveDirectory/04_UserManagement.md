# Lesson 4: User Management in Microsoft Entra ID

This is where you'll start working with Microsoft Entra ID in a practical way.

Every Azure administrator creates and manages users. Whether it's a new employee joining the company, a contractor needing temporary access, or an external partner collaborating with your organization, it all starts with **user identities**.

---

# Learning Objectives

By the end of this lesson, you will be able to:

* Understand what a User object is.
* Differentiate Member users and Guest users.
* Create, edit, delete, and restore users.
* Understand User Principal Name (UPN).
* Understand user properties.
* Manage passwords.
* Perform bulk user operations.
* Create users using the Azure Portal, Azure CLI, and PowerShell.

---

# What is a User?

A **User** is a digital identity stored in Microsoft Entra ID.

It represents a person who can sign in to Microsoft services and Azure resources.

Examples:

| Employee     | User Account                                                    |
| ------------ | --------------------------------------------------------------- |
| Raman Sharma | [raman@kmit.onmicrosoft.com](mailto:raman@kmit.onmicrosoft.com) |
| Amit Kumar   | [amit@kmit.onmicrosoft.com](mailto:amit@kmit.onmicrosoft.com)   |
| Priya Sharma | [priya@kmit.onmicrosoft.com](mailto:priya@kmit.onmicrosoft.com) |

A user object stores:

* Name
* Username
* Password (managed securely by Entra ID)
* Email
* Department
* Job Title
* Manager
* Groups
* Roles
* Licenses

---

# User Object

Think of a user as a record in the directory.

```text
User

├── Display Name
├── Username (UPN)
├── Email
├── Department
├── Job Title
├── Groups
├── Roles
├── Licenses
└── Authentication Methods
```

---

# Types of Users

There are two primary user types in Microsoft Entra ID.

## 1. Member User

A **Member** belongs to your organization.

Examples:

* Employees
* Administrators
* Developers
* HR staff

Example:

```text
raman@kmit.onmicrosoft.com
```

Member users are fully managed by your organization.

---

## 2. Guest User

A **Guest** is an external user invited into your tenant.

Example:

Your company collaborates with Microsoft.

You invite:

```text
john@microsoft.com
```

John remains a Microsoft employee but can access only the resources you've shared.

This is called **B2B Collaboration** (Business-to-Business).

Guest users are useful for:

* Vendors
* Consultants
* Customers
* Business partners

---

# Member vs Guest

| Member                            | Guest                                                         |
| --------------------------------- | ------------------------------------------------------------- |
| Internal employee                 | External user                                                 |
| Managed by your organization      | Managed by another organization or personal Microsoft account |
| Full identity in your tenant      | Limited identity in your tenant                               |
| Usually receives company licenses | Usually accesses shared resources only                        |

---

# User Principal Name (UPN)

The **UPN** is the user's login name.

Format:

```text
username@domain
```

Examples:

```text
raman@kmit.onmicrosoft.com

amit@contoso.com
```

It is similar to an email address but is primarily used for authentication.

---

# Display Name vs UPN

Example:

Display Name

```text
Raman Sharma
```

UPN

```text
raman@kmit.onmicrosoft.com
```

The Display Name is what users typically see in applications, while the UPN is used to sign in.

---

# Creating a User (Azure Portal)

Navigate to:

```text
Microsoft Entra ID

↓

Users

↓

New User
```

Choose:

* Create new user
* Invite external user (Guest)

Fill in:

* Username (UPN)
* Display Name
* Password
* Groups (optional)
* Roles (optional)

Click **Create**.

---

# Example

Display Name

```text
Raman Sharma
```

Username

```text
raman@kmit.onmicrosoft.com
```

Password

```text
Azure@12345
```

Entra ID creates the user object.

---

# User Properties

Each user has many properties.

Common ones include:

| Property        | Purpose              |
| --------------- | -------------------- |
| Display Name    | Friendly name        |
| Username (UPN)  | Login ID             |
| Job Title       | Employee designation |
| Department      | Organization unit    |
| Office Location | Office               |
| Manager         | Reporting manager    |
| Mobile Number   | Contact              |
| Email           | Mail address         |

These properties are commonly used in Microsoft 365, Teams, Outlook, and automation workflows.

---

# Reset Password

Administrators can reset a user's password.

Steps:

```text
Users

↓

Select User

↓

Reset Password
```

The administrator can:

* Generate a temporary password.
* Require the user to change it at the next sign-in.

---

# Delete a User

Steps:

```text
Users

↓

Select User

↓

Delete
```

The user is moved to the **Deleted Users** container.

---

# Restore a User

Deleted users are retained for a limited period (typically **30 days**).

Steps:

```text
Deleted Users

↓

Select User

↓

Restore
```

This restores the user's identity and many associated settings.

---

# Block Sign-In

Sometimes you don't want to delete a user.

Example:

An employee is on long leave.

Instead of deleting the account:

```text
User

↓

Block Sign-In
```

Benefits:

* User data remains intact.
* Access is immediately prevented.

This is often preferable to deletion for temporary situations.

---

# Bulk User Operations

Large organizations may need to create hundreds or thousands of users.

Instead of creating them one by one:

Use CSV import.

Example:

| Name  | Username                                                        | Department |
| ----- | --------------------------------------------------------------- | ---------- |
| Raman | [raman@kmit.onmicrosoft.com](mailto:raman@kmit.onmicrosoft.com) | IT         |
| Amit  | [amit@kmit.onmicrosoft.com](mailto:amit@kmit.onmicrosoft.com)   | HR         |
| Priya | [priya@kmit.onmicrosoft.com](mailto:priya@kmit.onmicrosoft.com) | Finance    |

Supported bulk operations include:

* Create users
* Delete users
* Invite guest users
* Update selected properties

---

# Azure CLI

Create a user:

```bash
az ad user create \
  --display-name "Raman Sharma" \
  --password "Azure@12345" \
  --user-principal-name raman@kmit.onmicrosoft.com \
  --force-change-password-next-login true
```

List users:

```bash
az ad user list --output table
```

Show a specific user:

```bash
az ad user show \
  --id raman@kmit.onmicrosoft.com
```

Delete a user:

```bash
az ad user delete \
  --id raman@kmit.onmicrosoft.com
```

> **Note:** The Azure CLI `az ad` commands use Microsoft Graph behind the scenes. In newer environments, Microsoft Graph PowerShell is often the preferred automation option for advanced identity management.

---

# PowerShell (Microsoft Graph)

Connect:

```powershell
Connect-MgGraph
```

Create a user:

```powershell
New-MgUser
```

Get users:

```powershell
Get-MgUser
```

Delete a user:

```powershell
Remove-MgUser
```

---

# Enterprise Scenario

A new employee joins KMIT.

Employee Details:

```text
Name: Rahul Verma

Department: IT

Job Title: DevOps Engineer
```

Administrator actions:

1. Create the user.
2. Assign an IT license.
3. Add to the Developers group.
4. Assign Azure permissions (if required).
5. Enable MFA.
6. Send temporary credentials.

The employee signs in and is ready to work.

---

# User Lifecycle

```text
Create User
      │
      ▼
Assign Groups
      │
      ▼
Assign License
      │
      ▼
Enable MFA
      │
      ▼
User Works
      │
      ▼
Block Sign-In (if needed)
      │
      ▼
Delete User
      │
      ▼
Restore (within retention period, if required)
```

---

# Hands-on Lab

## Lab 1: Create a Member User

Create a user with:

* Display Name: Rahul Verma
* Username: [rahul@yourtenant.onmicrosoft.com](mailto:rahul@yourtenant.onmicrosoft.com)
* Department: IT
* Job Title: DevOps Engineer

---

## Lab 2: Update User Information

Modify:

* Department
* Job Title
* Office Location
* Mobile Number

---

## Lab 3: Reset Password

Reset the user's password.

Enable:

* Require password change on next sign-in.

---

## Lab 4: Block and Unblock Sign-In

* Block the user from signing in.
* Verify the setting.
* Re-enable sign-in.

---

## Lab 5: Delete and Restore

1. Delete the user.
2. Open **Deleted Users**.
3. Restore the user.
4. Confirm the user appears again under **Users**.

---

# Interview Questions

1. What is a User object in Microsoft Entra ID?
2. What is the difference between a Member user and a Guest user?
3. What is a User Principal Name (UPN)?
4. How do you reset a user's password?
5. When would you block sign-in instead of deleting a user?
6. What are bulk user operations used for?
7. Where can you restore a deleted user?
8. What information is stored in a user object?

---

# Practice Assignment

1. Create **three Member users**:

   * DevOps Engineer
   * Azure Administrator
   * HR Manager

2. Update their:

   * Department
   * Job Title
   * Office Location

3. Reset the password for one user and require a password change at the next sign-in.

4. Block sign-in for one user, then unblock it.

5. Delete one user and restore it.

---

## What's Next?

In **Lesson 5: Groups in Microsoft Entra ID**, you'll learn:

* Security Groups
* Microsoft 365 Groups
* Assigned Membership
* Dynamic Membership
* Group-Based Licensing
* Group-Based Role Assignment
* Nested Groups (where applicable)
* Real-world enterprise access management using groups

This is where you'll see why organizations assign permissions to **groups** instead of directly to individual users, making administration much more scalable and secure.
