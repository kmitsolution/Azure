# What Happens When You Create an Azure Account?

When you sign up for Azure:

1. An **Azure Entra ID Tenant** (formerly Azure Active Directory) is created.
2. The tenant gets:

   * A **Tenant ID** (GUID)
   * A default domain name such as:

     ```text
     companyname.onmicrosoft.com
     ```
3. One or more **Azure Subscriptions** are associated with that tenant.
4. Inside a subscription, you create **Resource Groups**.
5. Inside Resource Groups, you deploy Azure Resources.

---

# Complete Azure Hierarchy

```text
Azure Account
(Email used for signup)
       │
       ▼
Azure Entra ID Tenant
(Azure AD)
       │
       ├── Tenant ID
       │
       └── Primary Domain
           companyname.onmicrosoft.com
       │
       ▼
Azure Subscription
(Pay-As-You-Go / Free Trial / Enterprise)
       │
       ├── Subscription ID
       ├── Billing
       ├── RBAC
       └── Quotas & Limits
       │
       ▼
Resource Groups
       │
       ├── RG-Production
       ├── RG-Dev
       └── RG-Test
       │
       ▼
Resources
       │
       ├── Virtual Machines
       ├── Storage Accounts
       ├── Virtual Networks
       ├── Key Vaults
       ├── Databases
       ├── Load Balancers
       └── App Services
```

---

# Real Example

Suppose you create an Azure account using:

```text
raman@gmail.com
```

Azure may create:

```text
Tenant Name:
ramantraining.onmicrosoft.com

Tenant ID:
12345678-1234-1234-1234-123456789abc
```

Inside that tenant:

```text
Pay-As-You-Go Subscription

Subscription ID:
87654321-4321-4321-4321-cba987654321
```

Inside the subscription:

```text
RG-Production
RG-Development
```

Inside RG-Production:

```text
VM-Web01
StorageAccount01
VNet-Prod
KeyVault-Prod
```

---

# Tenant vs Subscription

Many beginners confuse these.

## Tenant

Think of it as the **Identity and Security Boundary**

Contains:

* Users
* Groups
* Service Principals
* Applications
* Authentication
* MFA

Example:

```text
Tenant
│
├── Admin User
├── Developers
├── Applications
└── Security Policies
```

---

## Subscription

Think of it as the **Billing and Resource Boundary**

Contains:

* Azure Resources
* Cost Management
* Resource Groups
* Quotas

Example:

```text
Subscription
│
├── Billing
├── Resources
├── Resource Groups
└── RBAC
```

---

# One Tenant Can Have Multiple Subscriptions

Very common in enterprises.

```text
Azure Entra ID Tenant
│
├── Production Subscription
├── Development Subscription
├── Testing Subscription
└── Sandbox Subscription
```

All subscriptions use the same users from the tenant.

---

# One Subscription Can Have Multiple Resource Groups

```text
Production Subscription
│
├── RG-Network
├── RG-Web
├── RG-Database
└── RG-Monitoring
```

---

# Resource Group Contains Resources

```text
RG-Web
│
├── VM-Web01
├── VM-Web02
├── Load Balancer
└── Public IP
```

---

# Complete Enterprise Example

```text
Azure Account
(ITAdmin@company.com)
        │
        ▼
Azure Entra ID Tenant
(company.onmicrosoft.com)
        │
        ▼
+--------------------------------+
| Production Subscription        |
+--------------------------------+
        │
        ├── RG-Network
        │      ├── VNet
        │      └── NSG
        │
        ├── RG-Web
        │      ├── VM1
        │      ├── VM2
        │      └── Load Balancer
        │
        └── RG-Database
               └── SQL Database


+--------------------------------+
| Development Subscription       |
+--------------------------------+
        │
        ├── RG-Dev-Web
        └── RG-Dev-DB


+--------------------------------+
| Test Subscription              |
+--------------------------------+
        │
        └── RG-Test
```

---

# AZ-104 Exam Memory Trick

```text
Account
   ↓
Tenant
   ↓
Subscription
   ↓
Resource Group
   ↓
Resources
```

Remember:

* **Tenant** = Identity & Authentication
* **Subscription** = Billing & Resource Container
* **Resource Group** = Logical Grouping
* **Resource** = Actual Azure Service (VM, Storage, VNet, etc.)

### One-line exam answer

> An Azure account creates or associates with an Azure Entra ID tenant. The tenant provides identity management, subscriptions provide billing and resource boundaries, resource groups logically organize resources, and resources are the actual Azure services deployed within resource groups.
