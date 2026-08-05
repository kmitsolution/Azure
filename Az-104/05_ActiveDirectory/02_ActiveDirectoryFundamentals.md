# Lesson 2: Active Directory Fundamentals (On-Premises Active Directory)

Before we learn Microsoft Entra ID in depth, it's important to understand **Windows Active Directory (AD DS)** because Microsoft Entra ID evolved from the need to manage identities, but it is **not** the same product.

Many companies still use **Hybrid Identity**, where on-premises Active Directory and Microsoft Entra ID work together.

---

# Learning Objectives

By the end of this lesson, you will be able to:

* Understand why Active Directory was created.
* Explain the purpose of a Domain Controller.
* Understand Domains, Trees, Forests, and Organizational Units (OUs).
* Explain LDAP, Kerberos, DNS, and Group Policy.
* Understand how users authenticate in an Active Directory environment.
* Compare Active Directory with Microsoft Entra ID at a high level.

---

# Why Was Active Directory Created?

Imagine a company with **2,000 employees**.

Without Active Directory:

* Every PC has its own local users.
* Passwords differ on each computer.
* Administrators manage each PC separately.
* Employees can't easily use another computer.
* Access control becomes difficult.

Example:

| Computer | Username | Password |
| -------- | -------- | -------- |
| PC-01    | raman    | abc123   |
| PC-02    | raman    | xyz789   |
| PC-03    | raman    | pass123  |

This doesn't scale.

Microsoft created **Active Directory** so organizations could manage users, computers, and permissions from a central location.

---

# What is Active Directory?

**Active Directory Domain Services (AD DS)** is Microsoft's on-premises directory service.

It stores information about:

* Users
* Computers
* Groups
* Printers
* Servers
* Policies
* Organizational Units (OUs)

It provides:

* Authentication
* Authorization
* Centralized management
* Group Policy
* Single Sign-On (within the domain)

---

# What is a Domain?

A **Domain** is a logical boundary that groups users, computers, and resources under one administrative control.

Example:

```
Company: KMIT Solutions

Domain:

kmit.local
```

Everything belongs to this domain:

```
kmit.local

├── User1
├── User2
├── PC01
├── PC02
├── File Server
└── Printer
```

When a user logs into any domain-joined computer, they use the same domain credentials.

Example:

```
Username:

raman@kmit.local

Password:

********
```

The credentials are verified by a Domain Controller.

---

# What is a Domain Controller (DC)?

The **Domain Controller** is the most important server in Active Directory.

It stores the Active Directory database and performs:

* User authentication
* Password verification
* Group membership checks
* Computer authentication
* Security policy enforcement
* Replication with other Domain Controllers

Think of it as the "brain" of the Active Directory environment.

```
                Domain Controller
              +-------------------+
              | Active Directory  |
              | Users             |
              | Groups            |
              | Computers         |
              | Policies          |
              +-------------------+
```

---

# What Happens During Login?

Suppose Raman logs in to a company computer.

```
Username:

raman

Password:

********
```

The flow is:

```
User
   │
   ▼
Computer
   │
   ▼
Domain Controller
   │
   ▼
Verify Password
   │
   ▼
Authentication Successful
   │
   ▼
Desktop Opens
```

If the password is incorrect:

```
Access Denied
```

---

# What is an Organizational Unit (OU)?

An **Organizational Unit (OU)** is like a folder inside Active Directory.

It helps organize objects.

Example company:

```
KMIT

├── HR
├── IT
├── Finance
├── Sales
└── Management
```

Each OU can contain:

* Users
* Computers
* Groups

Example:

```
IT OU

├── Raman
├── Amit
├── Laptop01
└── Server01
```

Why use OUs?

* Organize users
* Delegate administration
* Apply different Group Policies

---

# What is a Tree?

A **Tree** is a collection of related domains that share a contiguous namespace.

Example:

```
kmit.com

├── training.kmit.com
├── cloud.kmit.com
└── hr.kmit.com
```

All domains are related.

---

# What is a Forest?

A **Forest** is the highest-level structure in Active Directory.

It can contain one or more trees.

Example:

```
Forest

├── kmit.com
│     ├── hr.kmit.com
│     └── training.kmit.com
│
└── company2.com
      └── sales.company2.com
```

A forest shares:

* Schema
* Global Catalog
* Trust relationships

A company can have one or multiple forests depending on business needs.

---

# What is DNS?

**DNS (Domain Name System)** translates names into IP addresses.

Example:

```
dc01.kmit.local

↓

192.168.1.10
```

Active Directory relies heavily on DNS to locate Domain Controllers and other services.

Without proper DNS, domain logons and many AD functions will fail.

---

# What is LDAP?

**LDAP (Lightweight Directory Access Protocol)** is used to query and manage directory information.

Applications use LDAP to:

* Search for users
* Find groups
* Retrieve user information

Example:

An HR application may ask:

> "Find the user Raman Sharma."

The application sends an LDAP query to Active Directory, which returns the user's details.

Think of LDAP as the language used to communicate with the directory.

---

# What is Kerberos?

**Kerberos** is the default authentication protocol used by Active Directory.

Instead of repeatedly sending passwords, Kerberos uses **tickets**.

Simplified flow:

```
User

↓

Domain Controller

↓

Ticket Issued

↓

Access File Server

↓

Ticket Verified

↓

Access Granted
```

Benefits:

* More secure than repeatedly sending passwords.
* Supports Single Sign-On (SSO).
* Faster authentication after initial login.

---

# What is Group Policy (GPO)?

A **Group Policy Object (GPO)** lets administrators apply settings to users and computers centrally.

Examples:

* Disable USB ports.
* Set desktop wallpaper.
* Configure password policies.
* Install software automatically.
* Prevent Control Panel access.
* Map network drives.

Without GPO, these settings would need to be configured manually on every computer.

---

# Active Directory Authentication Flow

```
User
   │
   ▼
Domain-Joined Computer
   │
   ▼
DNS locates Domain Controller
   │
   ▼
Kerberos authenticates user
   │
   ▼
Active Directory checks:
   • Username
   • Password
   • Group Membership
   │
   ▼
Group Policies Applied
   │
   ▼
Desktop Opens
```

---

# Active Directory vs Microsoft Entra ID

| Active Directory (AD DS)             | Microsoft Entra ID                                             |
| ------------------------------------ | -------------------------------------------------------------- |
| On-premises                          | Cloud-based                                                    |
| Uses Domain Controllers              | Managed by Microsoft                                           |
| Uses Kerberos & LDAP                 | Uses modern protocols like OAuth 2.0, OpenID Connect, and SAML |
| Supports Group Policy                | Uses Conditional Access and cloud management                   |
| Best for Windows domain environments | Best for cloud apps and Azure resources                        |
| Requires servers                     | No domain controllers to manage                                |

**Important:** Microsoft Entra ID is **not** simply "Active Directory in the cloud." It is a different identity platform designed for cloud services and modern authentication.

---

# Hands-on Lab

If you have access to Windows Server (physical, virtual, or in Azure), you can build a simple Active Directory lab:

1. Install Windows Server.
2. Assign a static IP address.
3. Install the **Active Directory Domain Services (AD DS)** role.
4. Promote the server to a Domain Controller.
5. Create a new forest with the domain:

   ```
   kmit.local
   ```
6. Create OUs:

   * HR
   * IT
   * Finance
7. Create a few users in each OU.
8. Join a Windows client machine to the domain.
9. Log in with a domain user account.
10. Create and apply a basic Group Policy (for example, change the desktop wallpaper).

---

# Interview Questions

1. What is Active Directory Domain Services (AD DS)?
2. What is the purpose of a Domain Controller?
3. What is the difference between a Domain and a Forest?
4. Why are Organizational Units (OUs) used?
5. What role does DNS play in Active Directory?
6. What is LDAP used for?
7. Why does Active Directory use Kerberos?
8. What is a Group Policy Object (GPO)?
9. How is Active Directory different from Microsoft Entra ID?

---

# Practice Assignment

1. Draw the hierarchy of an Active Directory environment showing:

   * Forest
   * Tree
   * Domain
   * OU
   * Users
   * Computers

2. Explain, in your own words, the responsibilities of:

   * Domain Controller
   * DNS
   * LDAP
   * Kerberos
   * Group Policy

3. Compare Active Directory and Microsoft Entra ID by listing at least **five differences**.

---

### Next Lesson

In **Lesson 3: Microsoft Entra ID Architecture**, we'll move into the cloud and cover:

* What is a **Tenant**?
* What is a **Directory** in Microsoft Entra ID?
* Azure Account vs Tenant vs Subscription
* How Azure resources trust Microsoft Entra ID
* Identity objects in Entra ID
* A complete sign-in flow from user login to Azure resource access

This lesson forms the foundation for everything you'll do in Azure identity management.
