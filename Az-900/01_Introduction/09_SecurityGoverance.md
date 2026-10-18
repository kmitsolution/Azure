
# AZ-900: Security and Governance Benefits in Azure

## 1. Introduction

Microsoft Azure provides several security and governance capabilities that help organizations protect their resources, manage access, enforce policies, meet compliance requirements, and control costs.

Security and governance are important topics in the Microsoft Azure Fundamentals (AZ-900) certification exam.

### Learning Objectives

By the end of this lesson, you will understand:

- Security in Azure
- Governance in Azure
- Microsoft Entra ID
- Multi-Factor Authentication (MFA)
- Role-Based Access Control (RBAC)
- Defense in Depth
- Zero Trust
- Azure Policy
- Resource Locks
- Resource Tags
- Microsoft Defender for Cloud
- Cost Management and compliance

---

## 2. What Is Security in Azure?

Security refers to protecting Azure resources, applications, data, and identities from unauthorized access, attacks, and other threats.

Azure provides multiple security services and features to help organizations protect their cloud environments.

### Examples

- Prevent unauthorized users from accessing Azure resources.
- Require additional authentication using MFA.
- Restrict access to virtual machines and storage accounts.
- Detect security threats and vulnerabilities.
- Protect sensitive data using encryption.

### Important Azure Security Services

| Service or Feature | Purpose |
|---|---|
| Microsoft Entra ID | Identity and access management |
| Multi-Factor Authentication (MFA) | Adds an additional authentication factor |
| Azure RBAC | Controls access to Azure resources |
| Microsoft Defender for Cloud | Security posture management and threat protection |
| Azure Key Vault | Stores secrets, keys, and certificates |
| Azure Firewall | Provides centralized network traffic filtering |
| Microsoft Sentinel | Security information and event management (SIEM) and security orchestration, automation, and response (SOAR) |

---

## 3. Microsoft Entra ID

Microsoft Entra ID, formerly known as Azure Active Directory (Azure AD), is Microsoft's cloud-based identity and access management service.

It helps organizations manage users, groups, applications, and authentication.

### Example Scenario

A company has 500 employees who need access to Azure resources.

The administrator can use Microsoft Entra ID to:

1. Create user accounts.
2. Organize users into groups.
3. Configure authentication requirements.
4. Enable MFA.
5. Manage access to supported applications and resources.

### Key Benefits

- Centralized identity management
- Single sign-on (SSO)
- Multi-Factor Authentication
- Conditional Access
- Identity-based access control

**AZ-900 exam tip:** Microsoft Entra ID primarily manages identities and authentication. Azure RBAC is used to authorize actions on Azure resources.

---

## 4. Multi-Factor Authentication (MFA)

Multi-Factor Authentication requires users to provide additional verification beyond their primary authentication method.

For example, a user may need a password and an approval through an authenticator application.

### Common Authentication Factors

- Something you know: Password or PIN
- Something you have: Authenticator device or security key
- Something you are: Fingerprint or facial recognition

### Example

A user enters their username and password to sign in to Azure.

The organization requires MFA, so the user must also approve a sign-in request or provide another supported verification method.

Even if an attacker obtains the password, the additional authentication factor provides another layer of protection.

### Benefits

- Reduces the risk of account compromise.
- Provides stronger authentication.
- Helps protect privileged accounts.
- Supports organizational security requirements.

---

## 5. Role-Based Access Control (RBAC)

Azure Role-Based Access Control is used to manage who can access Azure resources and which actions they can perform.

RBAC assignments are based on:

- Security principal: Who receives access, such as a user, group, or service principal.
- Role definition: What actions are allowed.
- Scope: Where the permissions apply.

### Common Built-In Roles

| Role | Permissions |
|---|---|
| Owner | Manage resources and assign access |
| Contributor | Create and manage resources, but cannot assign Azure RBAC roles |
| Reader | View resources but cannot make changes |
| User Access Administrator | Manage access to Azure resources |

### Example Scenario

A company has three employees:

- Administrator: Needs to manage resources and access.
- Developer: Needs to create and modify virtual machines.
- Auditor: Needs to view resources only.

The administrator can assign:

- Owner to the administrator.
- Contributor to the developer.
- Reader to the auditor.

In a real environment, assign the least privilege necessary and prefer narrower scopes.

### RBAC Scope Levels

Azure RBAC assignments can apply at different scopes:

1. Management group
2. Subscription
3. Resource group
4. Individual resource

Permissions generally inherit from parent scopes to child scopes.

**AZ-900 exam tip:** Follow the principle of least privilege by granting only the permissions required to perform a task.

---

## 6. Defense in Depth

Defense in Depth is a security strategy that uses multiple layers of protection.

If one security layer fails, other layers can help protect the environment.

### Common Security Layers

| Layer | Example |
|---|---|
| Physical security | Datacenter access controls |
| Identity and access | MFA and RBAC |
| Perimeter | DDoS protection and perimeter filtering |
| Network | Network Security Groups and Azure Firewall |
| Compute | Operating system updates and endpoint protection |
| Application | Secure development and application testing |
| Data | Encryption and access controls |

### Example Scenario

Consider a virtual machine hosting a company application.

The organization can protect it using:

1. MFA for administrators.
2. RBAC to restrict resource management.
3. NSG rules to restrict network traffic.
4. Operating system security updates.
5. Application security controls.
6. Encryption to protect sensitive data.

These controls provide multiple layers of security.

---

## 7. Zero Trust

Zero Trust is a security approach based on the principle:

**Never trust implicitly; always verify.**

It assumes that no user, device, or network connection should automatically be trusted.

### Core Principles

- Verify explicitly.
- Use least-privilege access.
- Assume breach.

### Example

An employee tries to access a sensitive application from a new device.

The organization may evaluate the user's identity, device compliance, location, and other available signals before allowing access.

Microsoft Entra Conditional Access can help enforce these access decisions.

### Benefits

- Reduces unauthorized access.
- Limits the impact of compromised accounts.
- Supports secure access from different locations.
- Helps protect applications and data.

---

## 8. Azure Governance

Governance refers to establishing and enforcing rules that help ensure Azure resources are deployed, configured, and managed according to organizational requirements.

Governance helps organizations maintain consistency, security, compliance, and cost control.

### Common Azure Governance Tools

| Tool | Purpose |
|---|---|
| Azure Policy | Enforces or audits organizational rules |
| Resource Locks | Helps prevent accidental deletion or modification |
| Resource Tags | Organizes resources using metadata |
| Management Groups | Organizes subscriptions into a hierarchy |
| Azure Cost Management | Analyzes and manages cloud spending |
| Microsoft Purview | Supports data governance and compliance capabilities |

---

## 9. Azure Policy

Azure Policy helps organizations enforce or audit rules for Azure resources.

Policies can evaluate resources to determine whether they comply with defined requirements.

### Example Scenario

A company wants all virtual machines to be deployed only in approved Azure regions.

An administrator can assign a policy that restricts deployments to those regions.

Another policy could audit whether resources have required tags, such as `Environment` or `Owner`.

### Common Policy Effects

| Effect | Purpose |
|---|---|
| Audit | Records noncompliance without blocking the resource operation |
| Deny | Blocks resource creation or updates that violate the policy |
| Append | Adds specified fields to a resource request |
| Modify | Modifies resource properties during applicable operations |
| DeployIfNotExists | Deploys related resources when specified conditions are met |
| AuditIfNotExists | Audits whether related resources exist |

The available effects depend on the policy definition and the resource operation.

### Important Point

Azure Policy can help enforce organizational standards, such as:

- Allowed Azure regions
- Required resource tags
- Permitted virtual machine configurations
- Required diagnostic settings

**AZ-900 exam tip:** Use Azure Policy when the requirement is to enforce or audit organizational rules.

---

## 10. Resource Locks

Resource Locks help protect Azure resources from accidental deletion or modification.

Azure provides two common lock types.

### Delete Lock

The `CanNotDelete` lock allows users to read and modify a resource but prevents its deletion while the lock applies.

### Read-Only Lock

The `ReadOnly` lock allows users to read a resource but prevents supported write and delete operations while the lock applies.

### Comparison

| Lock Type | Read | Modify | Delete |
|---|---|---|---|
| CanNotDelete | Yes | Yes | No |
| ReadOnly | Yes | No | No |

These restrictions apply while the lock is in effect. An authorized user with sufficient permissions may be able to remove a lock.

### Example Scenario

A company has a production database that must not be accidentally deleted.

The administrator applies a `CanNotDelete` lock to the database resource.

Users can continue to read and modify the resource, but deletion is blocked until the lock is removed.

**AZ-900 exam tip:** Use `CanNotDelete` to protect against accidental deletion and `ReadOnly` to prevent supported modifications and deletions.

---

## 11. Resource Tags

Tags are name-value pairs assigned to Azure resources.

They help organizations organize resources, identify ownership, categorize environments, and analyze costs.

### Example

| Resource | Environment | Department | Owner |
|---|---|---|---|
| VM-Web-01 | Production | IT | Team-A |
| VM-Dev-01 | Development | Development | Team-B |
| Storage-01 | Production | Finance | Team-C |

Example tags:

- `Environment = Production`
- `Department = Finance`
- `Owner = Team-A`
- `Project = ECommerce`

### Benefits

- Helps identify resource ownership.
- Supports cost analysis by department or project.
- Helps organize resources.
- Supports governance and reporting.

**Important:** Tags do not automatically enforce security permissions. Use RBAC for access control and Azure Policy to enforce required tagging rules.

---

## 12. Microsoft Defender for Cloud

Microsoft Defender for Cloud helps organizations assess and improve their security posture and provides threat protection capabilities for supported environments.

### Key Capabilities

- Security recommendations
- Security posture assessment
- Regulatory compliance views
- Threat protection for supported workloads
- Identification of security misconfigurations

### Example Scenario

A company deploys several virtual machines and storage accounts in Azure.

Defender for Cloud may identify security recommendations, such as improving network exposure or addressing supported workload vulnerabilities.

The security team can review these recommendations and take corrective action.

**AZ-900 exam tip:** Defender for Cloud is associated with cloud security posture management and workload protection.

---

## 13. Encryption and Azure Key Vault

Encryption helps protect data from unauthorized disclosure.

Azure supports encryption capabilities for data at rest and data in transit, depending on the service and configuration.

### Data at Rest

Data stored on disks, databases, or storage services can be protected using encryption.

### Data in Transit

Data moving between systems can be protected using secure communication protocols such as TLS.

### Azure Key Vault

Azure Key Vault securely stores and manages:

- Secrets
- Encryption keys
- Certificates

### Example

An application needs a database password.

Instead of storing the password directly in source code, the application can retrieve the secret from Key Vault using an appropriately configured identity and permissions.

This reduces the risk of exposing sensitive credentials in source control.

---

## 14. Compliance and Regulatory Requirements

Organizations may need to meet legal, industry, and internal compliance requirements.

Azure provides documentation, compliance offerings, and tools that help customers assess their cloud environments.

Examples of compliance frameworks and regulations include:

- ISO 27001
- SOC reports
- GDPR
- HIPAA, where applicable

The exact requirements depend on the organization's industry, location, data, and applicable laws.

**Important:** Using Azure does not automatically make every application or workload compliant. Customers must configure and operate their environments appropriately and meet their own compliance obligations.

---

## 15. Cost Governance

Cost governance helps organizations monitor, allocate, and control cloud spending.

Azure provides tools to help analyze costs and improve spending visibility.

### Azure Cost Management

Azure Cost Management helps users:

- Analyze resource spending.
- Review cost trends.
- Filter costs by subscription, resource group, service, or tag where supported.
- Create budgets.
- Configure cost alerts.

### Example Scenario

A company wants to keep its development environment spending within ₹20,000 per month.

The team can configure a budget and set alert thresholds to notify stakeholders when spending reaches specified levels.

A budget alert does not automatically stop all resources or guarantee that spending will remain below the budget.

### Cost Governance Best Practices

- Use resource tags for cost allocation.
- Remove unused resources.
- Right-size virtual machines.
- Review budgets and cost alerts.
- Monitor spending regularly.
- Use Azure pricing and cost calculators when planning deployments.

---

## 16. Security vs. Governance

| Feature | Main Purpose | Example |
|---|---|---|
| Microsoft Entra ID | Identity management | Manage users and sign-in |
| MFA | Stronger authentication | Require additional verification |
| Azure RBAC | Resource authorization | Give a developer Contributor access |
| Azure Policy | Enforce or audit rules | Restrict allowed regions |
| Resource Locks | Protect against accidental operations | Prevent database deletion |
| Resource Tags | Organize and categorize resources | Identify project ownership |
| Defender for Cloud | Security posture and threat protection | Review security recommendations |
| Azure Cost Management | Cost visibility and control | Create spending budgets |

---

## 17. Practical Scenario

### Scenario: Securing a Production Application

A company deploys a production application in Azure. It wants to protect the application, control access, enforce standards, and manage costs.

### Recommended Approach

1. Use Microsoft Entra ID to manage identities.
2. Require MFA for appropriate user and administrator sign-ins.
3. Assign Azure RBAC roles according to least privilege.
4. Use NSGs and other appropriate network controls.
5. Use Azure Policy to audit or enforce required configurations.
6. Apply a `CanNotDelete` lock to critical resources where appropriate.
7. Add tags such as `Environment`, `Owner`, and `Project`.
8. Use Defender for Cloud to review security recommendations.
9. Store application secrets in Azure Key Vault.
10. Configure budgets and cost alerts using Azure Cost Management.

This approach combines security, governance, and cost management to help operate Azure resources responsibly.

---

## 18. AZ-900 Exam Revision

Remember these key points:

- Microsoft Entra ID manages identities and authentication.
- MFA adds an additional authentication factor.
- Azure RBAC controls who can perform actions on Azure resources.
- Azure Policy enforces or audits organizational rules.
- Resource Locks help prevent accidental deletion or modification.
- `CanNotDelete` prevents deletion while the lock applies.
- `ReadOnly` prevents supported write and delete operations while the lock applies.
- Tags organize resources and support cost analysis.
- Microsoft Defender for Cloud helps assess security posture and protect supported workloads.
- Azure Key Vault manages secrets, keys, and certificates.
- Azure Cost Management helps analyze spending and configure budgets.
- Zero Trust follows the principles of verifying explicitly, using least privilege, and assuming breach.
- Defense in Depth uses multiple security layers.
- Azure compliance offerings do not automatically make every customer workload compliant.

---

## 19. Practice Quiz

Choose the correct answer for each question before checking the answer key.

### Question 1

Which Azure service is primarily used to manage identities and authentication?

A. Azure Policy  
B. Microsoft Entra ID  
C. Azure Cost Management  
D. Azure Monitor

### Question 2

A company wants to require users to provide a password and an additional verification factor. Which feature should it use?

A. Resource Tags  
B. Azure Policy  
C. Multi-Factor Authentication  
D. Resource Locks

### Question 3

Which Azure feature controls what actions a user can perform on Azure resources?

A. Azure RBAC  
B. Azure Key Vault  
C. Azure Cost Management  
D. Azure Advisor

### Question 4

An organization wants to restrict deployments to approved Azure regions. Which service should it use?

A. Azure Monitor  
B. Azure Policy  
C. Azure Key Vault  
D. Microsoft Entra ID

### Question 5

A company wants to prevent accidental deletion of a production resource while still allowing modifications. Which lock should it use?

A. ReadOnly  
B. CanNotDelete  
C. Contributor  
D. Reader

### Question 6

Which feature is most appropriate for identifying resources belonging to a particular department or project?

A. MFA  
B. Resource Tags  
C. Azure Firewall  
D. Microsoft Entra ID

### Question 7

Which Azure service helps assess security posture and provides security recommendations?

A. Microsoft Defender for Cloud  
B. Azure Cost Management  
C. Azure Policy only  
D. Azure DNS

### Question 8

An application needs to store database passwords securely rather than hardcoding them in source code. Which service is most appropriate?

A. Azure Load Balancer  
B. Azure Key Vault  
C. Azure Policy  
D. Azure Resource Manager

### Question 9

Which security approach assumes that access should be verified rather than trusted implicitly?

A. Consumption-based pricing  
B. Defense in Depth only  
C. Zero Trust  
D. Resource Tagging

### Question 10

A company wants to track Azure spending and receive alerts when spending reaches a specified threshold. Which tool should it use?

A. Azure Cost Management  
B. Microsoft Entra ID  
C. Azure Key Vault  
D. Azure Firewall

---

## 20. Quiz Answer Key with Explanations

1. **B — Microsoft Entra ID:** It manages identities and supports authentication and access-related capabilities.
2. **C — Multi-Factor Authentication:** MFA requires an additional authentication factor beyond the primary method.
3. **A — Azure RBAC:** RBAC determines which actions users and other security principals can perform at a given scope.
4. **B — Azure Policy:** Policies can audit or restrict resource deployments based on organizational requirements.
5. **B — CanNotDelete:** This lock prevents deletion while allowing supported read and modification operations.
6. **B — Resource Tags:** Tags provide name-value metadata for organizing resources and analyzing costs.
7. **A — Microsoft Defender for Cloud:** It provides security posture management and protection capabilities for supported workloads.
8. **B — Azure Key Vault:** Key Vault securely stores and manages secrets, keys, and certificates.
9. **C — Zero Trust:** Zero Trust requires explicit verification, least-privilege access, and an assume-breach mindset.
10. **A — Azure Cost Management:** It helps analyze spending, create budgets, and configure cost alerts.

---

## 21. Official Microsoft Learning Resources

- [AZ-900 Microsoft Azure Fundamentals Study Guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-900)
- [Azure Security Documentation](https://learn.microsoft.com/en-us/azure/security/)
- [Azure Role-Based Access Control](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)
- [Azure Policy Documentation](https://learn.microsoft.com/en-us/azure/governance/policy/overview)
- [Azure Resource Locks](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/lock-resources)
- [Azure Cost Management](https://learn.microsoft.com/en-us/azure/cost-management-billing/cost-management-billing-overview)

---

## Conclusion

Azure provides a broad set of security and governance capabilities to help organizations protect identities, control access, enforce standards, protect resources, meet compliance obligations, and manage spending.

For the AZ-900 exam, focus on understanding **what each service does, when to use it, and how it differs from similar services**.
