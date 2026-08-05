# Lesson 3: Microsoft Entra ID Architecture

This is one of the **most important lessons** in the entire Microsoft Entra ID course.

Almost every Azure interview includes questions such as:

* What is a Tenant?
* What is the relationship between a Tenant and a Subscription?
* Where are users stored?
* What is the difference between an Azure Account and an Entra ID Tenant?
* Can one Tenant have multiple subscriptions?
* Can one subscription belong to multiple tenants?

By the end of this lesson, you'll be understand these concepts clearly.

---

# Learning Objectives

After this lesson, you will be able to:

* Explain Microsoft Entra ID architecture.
* Understand Tenant, Directory, and Subscription.
* Explain the relationship between Azure Account, Tenant, and Subscription.
* Understand how authentication works in Azure.
* Navigate Microsoft Entra ID in the Azure Portal.
* Understand common enterprise architectures.

---

# Big Picture

When someone says:

> "I logged into Azure."

What actually happens?

Behind the scenes:

```text
User
   │
   ▼
Microsoft Entra ID
   │
Authenticate User
   │
   ▼
Azure Resource Manager (ARM)
   │
Check Permissions
   │
   ▼
Azure Resources
```

Notice something important:

**Every Azure login starts with Microsoft Entra ID.**

---

# What is Microsoft Entra ID?

Microsoft Entra ID is Microsoft's cloud Identity and Access Management (IAM) platform.

It manages:

* Users
* Groups
* Applications
* Devices
* Service Principals
* Managed Identities

It performs:

* Authentication
* Authorization
* Single Sign-On
* Multi-Factor Authentication (MFA)
* Conditional Access
* Identity Governance

Think of it as the **security gatekeeper** for Azure.

---

# What is a Tenant?

This is the most important concept.

A **Tenant** is a dedicated, isolated instance of Microsoft Entra ID for an organization.

Think of it as a company's private identity space.

Example:

```
Microsoft Cloud

│
├── Microsoft Tenant
│
├── Google Tenant
│
├── Amazon Tenant
│
└── KMIT Tenant
```

Each organization gets its own tenant.

Inside a tenant:

* Users
* Groups
* Applications
* Devices
* Roles
* Policies

are stored separately.

---

# Real-World Analogy

Imagine an apartment building.

The building is Microsoft Azure.

Each apartment is a Tenant.

```
Azure Building

Apartment 101 → Microsoft

Apartment 102 → Google

Apartment 103 → Amazon

Apartment 104 → KMIT
```

Residents in Apartment 104 cannot access Apartment 101 unless they are explicitly invited.

Similarly,

Each Tenant is isolated.

---

# What is a Directory?

In Microsoft Entra ID,

**Tenant** and **Directory** are often used interchangeably.

Technically,

A Directory is the database that stores identity objects.

It stores:

* Users
* Groups
* Applications
* Devices
* Service Principals
* Managed Identities

When Azure Portal says

```
Microsoft Entra ID
```

you are viewing your directory.

---

# Tenant Example

Suppose your company is KMIT.

Tenant Name

```
KMIT Solutions
```

Default Domain

```
kmitsolutions.onmicrosoft.com
```

Tenant ID

```
8f0b2a91-xxxx-xxxx-xxxx
```

Inside the tenant:

```
KMIT Tenant

├── Users
│     Raman
│     Amit
│     Priya
│
├── Groups
│     HR
│     Developers
│
├── Applications
│     Payroll App
│     CRM
│
├── Devices
│     Laptop01
│     Laptop02
│
└── Policies
```

---

# What is an Azure Account?

This is another commonly misunderstood concept.

Your Azure Account is **your login identity**.

Examples:

```
raman@gmail.com

raman@outlook.com

admin@company.com
```

You use this account to sign in.

After signing in,

Azure checks:

> Which Tenant can this account access?

One account can access:

* One Tenant
* Multiple Tenants

Example:

```
raman@gmail.com

↓

Can access

KMIT Tenant

Google Tenant

ABC Tenant
```

This is common for consultants and freelancers.

---

# Azure Account vs Tenant

| Azure Account               | Tenant                     |
| --------------------------- | -------------------------- |
| Login identity              | Organization               |
| Email address               | Company directory          |
| Used for login              | Stores identities          |
| Can access multiple tenants | Isolated identity boundary |

---

# What is a Subscription?

A Subscription is **not** an identity container.

It is a **billing and resource container**.

It contains Azure resources such as:

* Virtual Machines
* Storage Accounts
* Databases
* VNets
* App Services

Example:

```
Subscription

├── VM

├── Storage

├── SQL

├── App Service
```

Think of it as the container where you deploy and pay for Azure resources.

---

# Relationship Between Tenant and Subscription

This is one of the most frequently asked interview topics.

```
Tenant

↓

Subscription

↓

Resource Groups

↓

Resources
```

Example:

```
KMIT Tenant

│

├── Production Subscription

│      ├── VM

│      ├── Storage

│      └── SQL

│

└── Development Subscription

       ├── VM

       ├── AKS

       └── Storage
```

---

# Can One Tenant Have Multiple Subscriptions?

Yes.

Example:

```
KMIT Tenant

├── Production

├── Development

├── Testing

├── Sandbox

└── Training
```

This is very common.

---

# Can One Subscription Belong to Multiple Tenants?

**No.**

A Subscription can belong to **only one Tenant at a time**.

However,

You can **transfer** a subscription to another tenant if needed.

---

# Authentication Flow

Suppose Raman opens Azure Portal.

```
portal.azure.com

↓

Enter Username

↓

Enter Password

↓

Microsoft Entra ID

↓

Authentication

↓

Azure Resource Manager

↓

Check RBAC

↓

Access Azure Resources
```

Notice that **Microsoft Entra ID authenticates** the user, while **Azure Resource Manager (ARM)** enforces authorization using Azure RBAC.

---

# Enterprise Example

Suppose KMIT has:

Departments

* HR
* Finance
* IT
* Sales

The tenant contains:

```
Tenant

├── Users

├── Groups

├── Policies

├── Devices

├── Applications
```

Subscriptions

```
Production

Development

Testing
```

Resources

```
Production

├── VM

├── SQL

├── AKS

└── Storage
```

Users receive permissions based on Azure RBAC.

Example:

```
Raman

↓

Contributor

↓

Development Subscription
```

He cannot automatically manage the Production subscription unless granted access there.

---

# Azure Portal Walkthrough

Open:

```
portal.azure.com
```

Navigate to:

```
Microsoft Entra ID
```

Observe:

* Tenant Name
* Tenant ID
* Primary Domain
* License

Then go to:

* Users
* Groups
* App registrations
* Enterprise applications
* Devices
* Roles and administrators

Next, go to:

```
Subscriptions
```

Observe:

* Subscription Name
* Subscription ID
* Associated Tenant

---

# Common Interview Questions

### Q1. What is a Tenant?

A dedicated, isolated Microsoft Entra ID instance that stores an organization's identities, applications, devices, and policies.

---

### Q2. What is the difference between a Tenant and a Subscription?

| Tenant                        | Subscription                           |
| ----------------------------- | -------------------------------------- |
| Identity container            | Resource and billing container         |
| Stores users, groups, apps    | Stores Azure resources                 |
| Managed by Microsoft Entra ID | Managed through Azure Resource Manager |

---

### Q3. Can one Tenant have multiple subscriptions?

Yes.

---

### Q4. Can one subscription belong to multiple tenants?

No, only one tenant at a time.

---

### Q5. What authenticates users in Azure?

Microsoft Entra ID.

---

### Q6. What authorizes access to Azure resources?

Azure Resource Manager (ARM) using Azure Role-Based Access Control (RBAC).

---

# Hands-on Lab

## Lab 1: Explore Your Tenant

1. Sign in to the Azure Portal.
2. Open **Microsoft Entra ID**.
3. Record:

   * Tenant Name
   * Tenant ID
   * Primary Domain
4. Browse:

   * Users
   * Groups
   * Devices
   * Applications

## Lab 2: Explore Your Subscription

1. Search for **Subscriptions**.
2. Open your subscription.
3. Record:

   * Subscription Name
   * Subscription ID
   * Billing scope (if visible)
4. Open **Access control (IAM)** to see that permissions are managed at the subscription level using Azure RBAC.

---

# Key Takeaways

* **Microsoft Entra ID** is Azure's Identity and Access Management (IAM) service.
* A **Tenant** is an isolated identity boundary for an organization.
* A **Directory** stores users, groups, devices, applications, and policies; in Entra ID, "tenant" and "directory" are closely related concepts.
* An **Azure Account** is your sign-in identity.
* A **Subscription** is a billing and resource container.
* One **Tenant** can have many **Subscriptions**.
* One **Subscription** belongs to only one **Tenant** at a time.
* Authentication is performed by **Microsoft Entra ID**, while authorization to Azure resources is enforced by **Azure Resource Manager (ARM)** using **Azure RBAC**.

---

# Practice Assignment

Using your Azure subscription:

1. Find your:

   * Tenant Name
   * Tenant ID
   * Default Domain
   * Subscription Name
   * Subscription ID

2. Draw the following hierarchy from memory:

```text
Azure Account
      │
      ▼
Microsoft Entra ID Tenant
      │
      ▼
Subscription
      │
      ▼
Resource Group
      │
      ▼
Azure Resources
```

3. Explain, in your own words:

   * Why a subscription is **not** an identity container.
   * Why a tenant is **not** a billing container.
   * The roles of Microsoft Entra ID and Azure Resource Manager in the sign-in process.

In **Lesson 4**, we'll begin working directly with identities by creating and managing **Users** in Microsoft Entra ID, covering member users, guest users, bulk operations, user properties, and licensing with hands-on exercises in both the Azure portal and Azure CLI.
