## What is SKU in Azure?

**SKU (Stock Keeping Unit)** in Azure represents the **pricing tier, feature set, performance level, and capabilities** of an Azure resource.

Think of SKU as a **plan or edition** of a service.

For example, when creating a VM, Storage Account, or Load Balancer, Azure asks you to choose an SKU because different SKUs provide different:

* Performance
* Redundancy
* Features
* Scalability
* Pricing

---

## Example 1: Virtual Machine SKU

When creating a VM, you select a size such as:

* B2s
* D2s_v5
* E4s_v5

These are VM SKUs.

| SKU    | vCPU | RAM   | Suitable For          |
| ------ | ---- | ----- | --------------------- |
| B2s    | 2    | 4 GB  | Development/Test      |
| D2s_v5 | 2    | 8 GB  | General Purpose       |
| E4s_v5 | 4    | 32 GB | Memory Intensive Apps |

Example:

```bash
Standard_B2s
```

Here:

* **Standard** → Tier
* **B** → Burstable series
* **2** → Number of vCPUs
* **s** → Premium SSD support

---

## Example 2: Storage Account SKU

Storage Account SKUs determine replication and performance.

| SKU          | Meaning                    |
| ------------ | -------------------------- |
| Standard_LRS | Locally Redundant Storage  |
| Standard_GRS | Geo-Redundant Storage      |
| Standard_ZRS | Zone Redundant Storage     |
| Premium_LRS  | Premium SSD-backed Storage |

### Example

```bash
Standard_GRS
```

Means:

* Standard performance
* Data replicated to another Azure region

---

## Example 3: Load Balancer SKU

Azure Load Balancer has two major SKUs.

| SKU      | Features                 |
| -------- | ------------------------ |
| Basic    | Limited features         |
| Standard | Highly available, secure |

### Standard Load Balancer Features

* Availability Zone support
* Larger scale
* Better security
* Recommended for production

---

## Example 4: Public IP SKU

| SKU      | Characteristics               |
| -------- | ----------------------------- |
| Basic    | Dynamic by default            |
| Standard | Static by default, zone aware |

---

## Example 5: Azure SQL Database SKU

| SKU      | Use Case                   |
| -------- | -------------------------- |
| Basic    | Small workloads            |
| Standard | Medium workloads           |
| Premium  | High performance workloads |

---

## How to View Available SKUs using CLI

### VM SKUs

```bash
az vm list-skus --location centralindia -o table
```

### Storage Account SKUs

```bash
az storage account list-skus -o table
```

---

## Why Does Azure Use SKUs?

Because the same service may have multiple editions.

Example:

```text
Storage Account
      |
      +-- Standard_LRS
      +-- Standard_GRS
      +-- Standard_ZRS
      +-- Premium_LRS
```

Each SKU provides different:

* Cost
* Durability
* Features
* Performance

---

## Real-Life Analogy

Think of buying a car.

| Car Model   | Azure Equivalent |
| ----------- | ---------------- |
| Base Model  | Basic SKU        |
| Mid Variant | Standard SKU     |
| Top Variant | Premium SKU      |

Same product, but different capabilities and price.

---

## AZ-104 Exam Tip

Questions often ask:

> "Which SKU should be selected to provide zone redundancy?"

or

> "Which SKU supports Availability Zones?"

So always remember:

* **SKU = Pricing + Features + Performance + Capabilities**

### Examples to remember:

| Service       | Common SKUs              |
| ------------- | ------------------------ |
| VM            | B, D, E, F Series        |
| Storage       | LRS, GRS, ZRS            |
| Load Balancer | Basic, Standard          |
| Public IP     | Basic, Standard          |
| SQL           | Basic, Standard, Premium |

In simple words:

```text
SKU = Different editions/plans of an Azure service.
```
