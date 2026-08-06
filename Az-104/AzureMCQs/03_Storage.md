# AZ-104 Practice Questions

# Topic 2: Implement and Manage Storage (Questions 1–25)

> **Instructions:** Choose the best answer. Answers are provided at the end of the topic.

---

## Q1

Which Azure Storage service is designed for storing large amounts of unstructured data such as images, videos, and backups?

A. Azure Files

B. Blob Storage

C. Queue Storage

D. Table Storage

---

## Q2

Which storage redundancy option stores three copies of data within a single Azure region?

A. LRS

B. ZRS

C. GRS

D. RA-GRS

---

## Q3

Which storage redundancy replicates data across multiple Availability Zones in the same region?

A. LRS

B. ZRS

C. GRS

D. RA-GRS

---

## Q4

Which storage redundancy replicates data to a secondary Azure region?

A. LRS

B. ZRS

C. GRS

D. Premium SSD

---

## Q5

Which Blob Storage access tier is recommended for frequently accessed data?

A. Archive

B. Cool

C. Hot

D. Premium

---

## Q6

Which Blob Storage tier has the lowest storage cost but the highest retrieval cost?

A. Hot

B. Cool

C. Archive

D. Premium

---

## Q7

What is the primary purpose of an Azure Storage Account?

A. Host virtual machines

B. Store Azure storage services under one namespace

C. Create VNets

D. Manage Azure users

---

## Q8

Which storage service provides SMB file shares?

A. Blob Storage

B. Azure Files

C. Queue Storage

D. Table Storage

---

## Q9

Which protocol is commonly used by Azure Files?

A. HTTP

B. FTP

C. SMB

D. SSH

---

## Q10

Which storage service is optimized for storing messages between application components?

A. Blob Storage

B. Queue Storage

C. Azure Files

D. Disk Storage

---

## Q11

Which Azure Storage service is a NoSQL key-value store?

A. Azure Files

B. Queue Storage

C. Table Storage

D. Blob Storage

---

## Q12

Which feature automatically moves blobs between Hot, Cool, and Archive tiers?

A. Azure Backup

B. Lifecycle Management

C. Azure Monitor

D. Azure Policy

---

## Q13

Which authentication method provides temporary delegated access to storage resources?

A. Storage Account Key

B. Shared Access Signature (SAS)

C. Azure Policy

D. Azure Monitor

---

## Q14

Which type of SAS is secured using Microsoft Entra ID credentials?

A. Account SAS

B. Service SAS

C. User Delegation SAS

D. Anonymous SAS

---

## Q15

Which Azure feature encrypts data at rest by default?

A. Azure Backup

B. Storage Service Encryption (SSE)

C. Azure Monitor

D. Azure Firewall

---

## Q16

Which encryption key type is managed by Microsoft?

A. Customer-Managed Key (CMK)

B. Microsoft-Managed Key (MMK)

C. Bring Your Own Key (BYOK)

D. HSM Key

---

## Q17

Which Azure service is commonly used to store Customer-Managed Keys?

A. Azure Monitor

B. Azure Key Vault

C. Azure Policy

D. Azure Advisor

---

## Q18

Which Azure Storage feature enables point-in-time recovery for blobs?

A. Blob Versioning

B. Lifecycle Management

C. Queue Storage

D. Table Storage

---

## Q19

Which feature protects blobs from accidental deletion?

A. Soft Delete

B. Resource Lock

C. NSG

D. Azure Firewall

---

## Q20

Which Azure service synchronizes on-premises file servers with Azure Files?

A. Azure File Sync

B. Azure Backup

C. Azure Site Recovery

D. Azure Arc

---

## Q21

Which storage account type supports Blob, File, Queue, and Table services?

A. Premium Block Blob

B. FileStorage

C. StorageV2 (General-purpose v2)

D. BlobStorage

---

## Q22

Which Azure service is used to back up Azure file shares?

A. Recovery Services Vault

B. Azure Advisor

C. Azure Firewall

D. Application Gateway

---

## Q23

Which storage redundancy option allows read access to the secondary region?

A. LRS

B. GRS

C. RA-GRS

D. ZRS

---

## Q24

Which Azure Storage feature provides immutable storage for regulatory compliance?

A. Lifecycle Management

B. Immutable Blob Storage (WORM)

C. Queue Storage

D. Azure Files

---

## Q25

Which protocol is used to access Azure Blob Storage over the internet?

A. SMB

B. NFS

C. HTTPS

D. RDP

---

# Answer Key

| Q | Ans | Q  | Ans | Q  | Ans | Q  | Ans | Q  | Ans |
| - | --- | -- | --- | -- | --- | -- | --- | -- | --- |
| 1 | B   | 6  | C   | 11 | C   | 16 | B   | 21 | C   |
| 2 | A   | 7  | B   | 12 | B   | 17 | B   | 22 | A   |
| 3 | B   | 8  | B   | 13 | B   | 18 | A   | 23 | C   |
| 4 | C   | 9  | C   | 14 | C   | 19 | A   | 24 | B   |
| 5 | C   | 10 | B   | 15 | B   | 20 | A   | 25 | C   |

---

## Exam Tips (AZ-104)

* **LRS** = 3 copies in one datacenter.
* **ZRS** = 3 copies across Availability Zones in the same region.
* **GRS** = Replicates data to a paired Azure region.
* **RA-GRS** = GRS + read access to the secondary region.
* **Hot** = Frequently accessed data.
* **Cool** = Infrequently accessed data (minimum retention charges apply).
* **Archive** = Lowest storage cost, highest retrieval latency and cost.
* **SAS** = Temporary, time-limited access without sharing account keys.
* **User Delegation SAS** = Preferred when using Microsoft Entra ID.
* **StorageV2** is the recommended storage account type for most workloads.

These questions are written in the style of the AZ-104 exam and emphasize concepts that are commonly tested. The next set (Questions **26–50**) will cover **Azure Disks, Storage Security, Import/Export, AzCopy, Storage Explorer, Backup, Snapshots, and Storage Migration**.
