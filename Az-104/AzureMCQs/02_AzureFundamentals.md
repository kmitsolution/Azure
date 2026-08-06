# AZ-104 Practice Questions

# Topic 1: Manage Azure Identities and Governance (Questions 26–50)

> **Instructions:** Choose the best answer. Answers are at the bottom.

---

## Q26

You need to ensure that all newly created resources include the tag **Environment**. Which Azure service should you use?

A. Azure Advisor

B. Azure Policy

C. Azure Monitor

D. Resource Lock

---

## Q27

Which Azure feature allows a policy to automatically add a missing tag during resource creation?

A. Deny

B. Audit

C. Append

D. Modify

---

## Q28

A company wants to prevent users from creating resources outside the **East US** region. Which Azure feature should be used?

A. Azure Monitor

B. Azure Policy

C. Azure Backup

D. Azure Firewall

---

## Q29

Which built-in RBAC role allows managing role assignments without giving full ownership of resources?

A. Contributor

B. Reader

C. User Access Administrator

D. Virtual Machine Contributor

---

## Q30

A user should manage only virtual machines but not storage accounts. What is the best approach?

A. Assign Owner at the subscription level.

B. Assign Virtual Machine Contributor at the required scope.

C. Assign Reader at the subscription level.

D. Assign Contributor at the tenant level.

---

## Q31

Which Azure feature allows inviting external users to collaborate with your organization?

A. Azure Policy

B. Microsoft Entra B2B

C. Azure Backup

D. Azure Firewall

---

## Q32

Which Microsoft Entra ID edition includes Conditional Access?

A. Free

B. Microsoft Entra ID P1 or P2

C. Basic

D. Azure AD Connect

---

## Q33

What is the purpose of a Management Group?

A. Store virtual machines

B. Organize subscriptions and apply governance at scale

C. Replace Resource Groups

D. Manage storage accounts

---

## Q34

A policy is configured with the **Audit** effect. What happens when a non-compliant resource is created?

A. The deployment is blocked.

B. The resource is deleted.

C. The deployment succeeds, but the resource is marked as non-compliant.

D. The resource is automatically fixed.

---

## Q35

Which Azure feature is used to evaluate compliance with organizational standards?

A. Azure Policy

B. Azure Load Balancer

C. Azure Files

D. Azure Disk Encryption

---

## Q36

A Resource Group contains resources in:

A. Multiple Azure tenants

B. Multiple subscriptions

C. Different Azure regions (if supported)

D. Different Microsoft Entra tenants

---

## Q37

Which statement about Resource Groups is correct?

A. A Resource Group can contain resources from multiple subscriptions.

B. A resource can belong to multiple Resource Groups.

C. A Resource Group belongs to one subscription.

D. Resource Groups cannot be moved.

---

## Q38

What happens if you delete a Resource Group?

A. Only the Resource Group is deleted.

B. Only virtual machines are deleted.

C. All resources within the Resource Group are deleted.

D. Storage accounts remain.

---

## Q39

Which Azure feature helps identify cost allocation by department?

A. Tags

B. NSG

C. Azure Firewall

D. Availability Set

---

## Q40

Which RBAC scope provides permissions only to a single virtual machine?

A. Management Group

B. Subscription

C. Resource Group

D. Resource

---

## Q41

Which Azure identity type is recommended for applications running on Azure resources?

A. Local administrator account

B. Managed Identity

C. Shared administrator account

D. Storage account key

---

## Q42

Which feature provides **just-in-time** privileged role activation?

A. Azure Backup

B. Azure Advisor

C. Microsoft Entra Privileged Identity Management (PIM)

D. Azure Firewall

---

## Q43

Which Azure Policy effect blocks non-compliant resource deployments?

A. Audit

B. Append

C. Deny

D. Modify

---

## Q44

Which feature can automatically remediate non-compliant resources after deployment?

A. Modify Policy

B. Azure Monitor

C. Resource Lock

D. Availability Set

---

## Q45

Which RBAC assignment requires three components?

A. User, Subscription, Region

B. Security Group, VM, Disk

C. Security Principal, Role Definition, Scope

D. User, Storage Account, Resource Group

---

## Q46

Which Azure feature allows permissions to be inherited from a higher scope?

A. Azure Monitor

B. RBAC

C. Azure Backup

D. Azure Advisor

---

## Q47

A company wants all subscriptions to inherit the same policies. Where should the policies be assigned?

A. Resource

B. Resource Group

C. Subscription

D. Management Group

---

## Q48

Which identity can exist independently of an Azure resource?

A. System-assigned Managed Identity

B. User-assigned Managed Identity

C. Local Administrator

D. VM Extension

---

## Q49

What is the purpose of Microsoft Entra Connect?

A. Monitor Azure resources

B. Synchronize on-premises Active Directory with Microsoft Entra ID

C. Backup virtual machines

D. Manage Azure networking

---

## Q50

Which governance feature helps prevent accidental modification of critical resources?

A. Azure Advisor

B. Azure Policy

C. Read-only Resource Lock

D. Availability Zone

---

# Answer Key

| Q  | Ans | Q  | Ans | Q  | Ans | Q  | Ans | Q  | Ans |
| -- | --- | -- | --- | -- | --- | -- | --- | -- | --- |
| 26 | B   | 31 | B   | 36 | C   | 41 | B   | 46 | B   |
| 27 | D   | 32 | B   | 37 | C   | 42 | C   | 47 | D   |
| 28 | B   | 33 | B   | 38 | C   | 43 | C   | 48 | B   |
| 29 | C   | 34 | C   | 39 | A   | 44 | A   | 49 | B   |
| 30 | B   | 35 | A   | 40 | D   | 45 | C   | 50 | C   |

---

## Key Explanations

* **Q27:** **Modify** can add or update tags during or after deployment (with remediation). **Append** adds information during creation but doesn't remediate existing resources.
* **Q29:** **User Access Administrator** can manage RBAC role assignments without full resource ownership.
* **Q30:** Follow the **least privilege principle** by assigning the **Virtual Machine Contributor** role at the smallest required scope.
* **Q34:** **Audit** records non-compliance but does not block deployments.
* **Q43:** The **Deny** effect prevents non-compliant resources from being created.
* **Q45:** Every RBAC assignment consists of a **Security Principal**, a **Role Definition**, and a **Scope**.
* **Q47:** Assign policies at the **Management Group** level to have them inherited by all child subscriptions.
* **Q50:** A **Read-only Resource Lock** prevents modifications (and effectively prevents deletion because deletion requires a write operation).
