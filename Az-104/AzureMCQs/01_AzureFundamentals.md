
# AZ-104 Practice Questions

# Topic 1: Manage Azure Identities and Governance (Questions 1–25)

> **Instructions:** Choose the best answer. Answers are provided at the end of the topic.

---

### Q1

A user needs to manage Azure resources but should not be able to assign RBAC roles. Which built-in role should you assign?

A. Owner

B. Contributor

C. Reader

D. User Access Administrator

---

### Q2

Which Microsoft service is responsible for authentication and identity management in Azure?

A. Azure Policy

B. Microsoft Entra ID

C. Azure Monitor

D. Azure Backup

---

### Q3

Which Azure feature determines **who** can access Azure resources?

A. Azure Policy

B. Azure Advisor

C. Azure RBAC

D. Azure Monitor

---

### Q4

Which Azure service enforces compliance by restricting resource deployment?

A. RBAC

B. Azure Policy

C. Resource Lock

D. Microsoft Defender for Cloud

---

### Q5

Which RBAC role provides full access, including assigning roles?

A. Reader

B. Contributor

C. Owner

D. Security Reader

---

### Q6

Which RBAC role allows viewing resources but prevents any modifications?

A. Reader

B. Contributor

C. Owner

D. Virtual Machine Contributor

---

### Q7

A resource has a **Delete Lock** applied. What happens?

A. Resource cannot be modified.

B. Resource cannot be deleted.

C. Resource cannot be viewed.

D. Resource cannot be started.

---

### Q8

What is the primary purpose of Azure Tags?

A. Security

B. Cost management and organization

C. Backup

D. Authentication

---

### Q9

Which Azure component acts as the billing boundary?

A. Resource Group

B. Management Group

C. Subscription

D. Tenant

---

### Q10

A Resource Group can contain resources from:

A. Multiple subscriptions

B. Multiple tenants

C. Only one subscription

D. Unlimited tenants

---

### Q11

Which Azure feature allows grouping multiple subscriptions for governance?

A. Resource Group

B. Management Group

C. Azure Policy

D. Availability Set

---

### Q12

What is the highest scope in Azure RBAC?

A. Subscription

B. Resource Group

C. Management Group

D. Resource

---

### Q13

Which feature automatically inherits policies and RBAC assignments?

A. Resource Locks

B. Tags

C. Management Groups

D. Azure Backup

---

### Q14

Which Azure feature helps prevent accidental deletion?

A. RBAC

B. Azure Policy

C. Resource Lock

D. Azure Monitor

---

### Q15

Which lock prevents modification but allows deletion?

A. Delete Lock

B. Read-only Lock

C. Resource Lock

D. Security Lock

---

### Q16

Microsoft Entra ID was formerly known as:

A. Azure Security Center

B. Azure Active Directory

C. Azure Monitor

D. Microsoft Defender

---

### Q17

Which authentication method provides the strongest security?

A. Username only

B. Username and Password

C. Multi-Factor Authentication (MFA)

D. PIN only

---

### Q18

Which Azure feature evaluates user sign-in conditions before granting access?

A. Azure Policy

B. Conditional Access

C. Azure Backup

D. Azure Monitor

---

### Q19

Which scope is the narrowest in Azure RBAC?

A. Management Group

B. Subscription

C. Resource Group

D. Resource

---

### Q20

Which Azure feature allows temporary elevated access to privileged roles?

A. Azure Backup

B. Azure Advisor

C. Microsoft Entra Privileged Identity Management (PIM)

D. Azure Monitor

---

### Q21

What is the primary purpose of a Service Principal?

A. Manage billing

B. Represent an application or service for authentication

C. Create Resource Groups

D. Configure Availability Zones

---

### Q22

Which identity type is automatically deleted when its Azure resource is deleted?

A. User-assigned Managed Identity

B. System-assigned Managed Identity

C. Service Principal

D. Guest User

---

### Q23

Which identity can be shared across multiple Azure resources?

A. System-assigned Managed Identity

B. User-assigned Managed Identity

C. Local User

D. Azure Policy Identity

---

### Q24

What is the main benefit of Managed Identities?

A. Lower storage costs

B. Eliminate the need to store credentials in code

C. Improve VM performance

D. Increase bandwidth

---

### Q25

Which Azure feature is primarily used to enforce naming conventions and required tags?

A. Azure Policy

B. Azure Advisor

C. Azure Monitor

D. Azure Firewall

---

# Answer Key

| Q | Ans | Q  | Ans | Q  | Ans | Q  | Ans | Q  | Ans |
| - | --- | -- | --- | -- | --- | -- | --- | -- | --- |
| 1 | B   | 6  | A   | 11 | B   | 16 | B   | 21 | B   |
| 2 | B   | 7  | B   | 12 | C   | 17 | C   | 22 | B   |
| 3 | C   | 8  | B   | 13 | C   | 18 | B   | 23 | B   |
| 4 | B   | 9  | C   | 14 | C   | 19 | D   | 24 | B   |
| 5 | C   | 10 | C   | 15 | B   | 20 | C   | 25 | A   |

---

## Explanation Highlights

* **Q1:** **Contributor** can manage Azure resources but **cannot assign RBAC roles**.
* **Q4:** **Azure Policy** enforces organizational standards such as allowed regions, required tags, or approved VM sizes.
* **Q5:** **Owner** has full control, including the ability to assign RBAC roles.
* **Q7:** A **Delete Lock** prevents deletion but still allows updates to the resource.
* **Q8:** **Tags** help organize resources for billing, automation, and reporting.
* **Q12:** The highest RBAC scope is the **Management Group**, followed by Subscription, Resource Group, and Resource.
* **Q18:** **Conditional Access** evaluates conditions like user, location, device, or risk before allowing sign-in.
* **Q20:** **Microsoft Entra PIM** provides just-in-time, temporary elevation to privileged roles.
* **Q22 & Q23:** **System-assigned Managed Identities** are tied to a single resource and deleted with it, while **User-assigned Managed Identities** are standalone and can be shared across multiple resources.
* **Q25:** **Azure Policy** is commonly used to enforce naming conventions, required tags, and compliance requirements.


