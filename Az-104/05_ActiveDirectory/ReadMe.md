# Microsoft Entra ID (Azure AD) 

## Phase 1: Identity Fundamentals (Must Learn First)

### Lesson 1 – Introduction to Identity

Learn

- What is Identity?
- Authentication vs Authorization
- Identity Provider (IdP)
- Directory Service
- Active Directory
- Azure Active Directory
- Microsoft Entra ID
- Why Microsoft changed Azure AD to Entra ID
- Identity vs Account

Practical

- Create Azure Free Account
- Explore Microsoft Entra Admin Center

---

### Lesson 2 – Active Directory Basics

Learn

- On-Prem Active Directory
- Domain
- Forest
- Tree
- Organizational Unit (OU)
- Domain Controller
- LDAP
- Kerberos
- NTLM
- Group Policy

Understand

```
User
      |
      |
Domain Controller
      |
Active Directory Database
```

---

### Lesson 3 – Microsoft Entra ID Architecture

Learn

How Entra ID works.

Understand

```
User

↓

Microsoft Entra ID

↓

Authentication

↓

Azure Resource
```

Topics

- Tenant
- Directory
- Subscription
- Relationship between Tenant and Subscription
- Azure Account

Practical

Create

- One Tenant
- One Subscription
- Link both

---

# Phase 2: User Management

---

### Lesson 4 – Users

Learn

- Create Users
- Delete Users
- Restore Users
- Guest Users
- Member Users
- Bulk Import
- Bulk Delete
- Bulk Update

Lab

Create

```
Admin User

Developer

Tester

HR

Finance
```

---

### Lesson 5 – Groups

Learn

Group Types

- Security Group
- Microsoft 365 Group

Membership

- Assigned
- Dynamic User
- Dynamic Device

Lab

Create

```
Developers

HR

Managers

Admins
```

Assign Users

---

### Lesson 6 – Administrative Units

Learn

What are Administrative Units

Why companies use them

Lab

```
India

USA

Europe
```

Assign users accordingly

---

# Phase 3: Authentication

---

### Lesson 7 – Authentication Methods

Learn

Password

Password Hash

Passwordless

FIDO2

Windows Hello

Microsoft Authenticator

SMS

Voice Call

Temporary Access Pass

Passkeys

Lab

Enable each method

---

### Lesson 8 – Multi-Factor Authentication (MFA)

Learn

Why MFA

How MFA works

Authentication Flow

Lab

Enable MFA

Test Login

Disable MFA

Conditional MFA

---

### Lesson 9 – Self Service Password Reset (SSPR)

Learn

- Password Reset
- Password Writeback
- Security Questions
- Authentication Methods

Lab

Enable SSPR

Test Reset

---

# Phase 4: Authorization

---

### Lesson 10 – RBAC

Very Important

Learn

Built-in Roles

Examples

```
Global Administrator

User Administrator

Security Administrator

Billing Administrator

Application Administrator

Reader

Owner

Contributor
```

Lab

Assign

Reader

Contributor

Owner

Verify permissions

---

### Lesson 11 – Custom Roles

Learn

Role Definitions

Role Assignments

Scopes

Lab

Create Custom Role

Assign

Verify

---

# Phase 5: Enterprise Applications

---

### Lesson 12 – Enterprise Applications

Learn

What is Enterprise Application

Service Principal

Application Registration

Managed Identity

OAuth

OIDC

SAML

Lab

Register Application

Assign Permissions

Grant Consent

---

### Lesson 13 – App Registration

Very Important

Learn

Application

Client ID

Tenant ID

Object ID

Redirect URI

Certificates

Secrets

API Permissions

Expose API

Lab

Register Web App

Generate Secret

Test Authentication

---

# Phase 6: Identity Protection

---

### Lesson 14 – Conditional Access

One of the Most Important Topics

Learn

Policies

Conditions

Locations

Devices

Risk

Applications

Controls

Lab

Create Policy

```
Require MFA

Outside Office

Block Legacy Authentication

Only Windows Devices
```

---

### Lesson 15 – Identity Protection

Learn

Risk

User Risk

Sign-in Risk

Risk Detection

Policies

Lab

Review Risky Sign-ins

Review Risky Users

---

# Phase 7: Hybrid Identity

---

### Lesson 16 – Azure AD Connect

Learn

Synchronization

Password Hash Sync

Pass-through Authentication

Federation

Lab

Understand

```
On Prem AD

↓

Azure AD Connect

↓

Entra ID
```

---

### Lesson 17 – Hybrid Identity

Learn

Hybrid Join

Azure AD Join

Registered Devices

Domain Join

Device Writeback

---

# Phase 8: Devices

---

### Lesson 18 – Device Management

Learn

Azure AD Join

Hybrid Join

Registered Devices

Device Compliance

Intune Integration

---

# Phase 9: Governance

---

### Lesson 19 – Identity Governance

Learn

Entitlement Management

Access Reviews

Lifecycle Workflows

Privileged Identity Management (PIM)

Access Packages

---

# Phase 10: Security

---

### Lesson 20 – Microsoft Defender for Identity

Learn

Threat Detection

Lateral Movement

Identity Attack Detection

Investigation

---

# Phase 11: Monitoring

---

### Lesson 21 – Monitoring

Learn

Audit Logs

Sign-in Logs

Provisioning Logs

Workbooks

Diagnostic Settings

Log Analytics

---

# Phase 12: B2B and B2C

---

### Lesson 22 – External Identities

Learn

Guest Users

B2B Collaboration

Cross Tenant Access

Invitation Flow

---

### Lesson 23 – Customer Identity

Learn

Customer Authentication

External Identity

Consumer Login

Social Login

---

# Phase 13: Managed Identity

---

### Lesson 24 – Managed Identity

Learn

System Assigned

User Assigned

VM Identity

App Service Identity

Key Vault Authentication

Storage Authentication

Lab

Create

VM

Managed Identity

Access Storage

---

# Phase 14: OAuth and Tokens

---

### Lesson 25 – OAuth 2.0

Learn

Authorization Code

Client Credentials

Device Code

Implicit Flow (Legacy)

PKCE

Refresh Token

Access Token

ID Token

JWT

---

### Lesson 26 – OpenID Connect & SAML

Learn

OIDC

SAML

Claims

Federation

Single Sign-On (SSO)

---

# Phase 15: Enterprise Scenarios

### Lesson 27 – Real Company Scenarios

Build practical scenarios such as:

- New employee onboarding
- Employee resignation (offboarding)
- Temporary contractor access
- Department-based access using groups
- VM authentication using Managed Identity
- Web App authentication with Entra ID
- Secure Azure Storage access without connection strings
- Role-Based Access Control (RBAC) for Azure resources
- Single Sign-On (SSO) for third-party SaaS applications
- Multi-Factor Authentication (MFA) for administrators
- Conditional Access policies for remote users
- Privileged Identity Management (PIM) for just-in-time admin access

---

# Hands-on Labs

Throughout the course, you should complete these labs:

1. Create an Entra ID tenant.
2. Create users, groups, and administrative units.
3. Assign Azure RBAC roles at subscription and resource-group scope.
4. Configure authentication methods, MFA, and Self-Service Password Reset (SSPR).
5. Create dynamic groups using membership rules.
6. Register an application and authenticate using OAuth 2.0/OpenID Connect.
7. Configure Conditional Access policies.
8. Configure Microsoft Entra Connect in a hybrid identity lab (or understand the architecture if you don't have on-premises infrastructure).
9. Create and use Managed Identities with Azure VMs, App Services, and Key Vault.
10. Integrate an application with Microsoft Entra ID for Single Sign-On (SSO).
11. Monitor sign-in logs, audit logs, and identity-related events.

---

# Recommended Learning Order

| Phase | Topic | Difficulty |
|--------|-------|------------|
| 1 | Identity Fundamentals | ⭐ |
| 2 | Users & Groups | ⭐ |
| 3 | Authentication | ⭐⭐ |
| 4 | Authorization (RBAC) | ⭐⭐ |
| 5 | Enterprise Applications | ⭐⭐⭐ |
| 6 | Conditional Access & Identity Protection | ⭐⭐⭐ |
| 7 | Hybrid Identity | ⭐⭐⭐ |
| 8 | Device Identity | ⭐⭐⭐ |
| 9 | Governance (PIM, Access Reviews) | ⭐⭐⭐⭐ |
| 10 | Monitoring & Logs | ⭐⭐⭐ |
| 11 | External Identities (B2B/B2C) | ⭐⭐⭐ |
| 12 | Managed Identity | ⭐⭐⭐ |
| 13 | OAuth 2.0, OIDC & SAML | ⭐⭐⭐⭐ |
| 14 | Enterprise Scenarios | ⭐⭐⭐⭐ |

## My recommendation

Given your background in Azure networking, infrastructure, and Managed Identity, I'd teach this as a structured course with each lesson including:

1. Theory (concepts and terminology)
2. Azure portal demonstration
3. Azure CLI and PowerShell equivalents
4. Architecture diagrams
5. Real-world enterprise scenarios
6. Hands-on labs
7. Interview questions
8. Troubleshooting exercises
9. Practice assignments

This approach will prepare you not only for **AZ-104**, but also for **AZ-500 (Azure Security Engineer)**, **SC-300 (Microsoft Identity and Access Administrator)**, and real-world enterprise identity administration.

I recommend we start with **Lesson 1: Identity Fundamentals**, where we'll build the foundation by understanding identity, authentication, authorization, identity providers, directories, and why Microsoft Entra ID is central to Azure.
