Excellent. Let's do a **complete hands-on lab**. We'll first deploy Azure Firewall using the **Azure Portal**, then using the **Azure CLI**.

> **Lab Objective**
>
> Deploy Azure Firewall in a Hub VNet.

We will create the following architecture:

```text
                 Internet
                     │
              Public IP (Firewall)
                     │
             Azure Firewall
                     │
             AzureFirewallSubnet
                     │
                 Hub VNet
             10.0.0.0/16
```

---

# Part 1: Deploy Azure Firewall using Azure Portal

## Step 1: Create Resource Group

Navigate to:

```text
Azure Portal
    ↓
Resource Groups
    ↓
Create
```

Enter:

| Property       | Value            |
| -------------- | ---------------- |
| Resource Group | RG-Firewall-Demo |
| Region         | Central India    |

Click **Review + Create**.

---

# Step 2: Create Hub VNet

Navigate to:

```text
Virtual Networks
```

Click:

```text
Create
```

### Basics

| Property      | Value       |
| ------------- | ----------- |
| Name          | Hub-VNet    |
| Address Space | 10.0.0.0/16 |

---

### Create the Required Firewall Subnet

Azure Firewall **must** be deployed into a subnet named:

```text
AzureFirewallSubnet
```

This name is mandatory.

Create the subnet:

| Property       | Value               |
| -------------- | ------------------- |
| Name           | AzureFirewallSubnet |
| Address Prefix | 10.0.0.0/26         |

Minimum recommended size:

```text
/26
```

Click **Create**.

---

# Why AzureFirewallSubnet?

Azure Firewall creates multiple internal instances.

Microsoft reserves addresses inside this subnet.

Therefore:

```text
AzureFirewallSubnet
```

is mandatory.

---

# Step 3: Create Public IP

Navigate to:

```text
Public IP Addresses
```

Click:

```text
Create
```

Enter:

| Property   | Value         |
| ---------- | ------------- |
| Name       | Firewall-PIP  |
| SKU        | Standard      |
| Assignment | Static        |
| Region     | Central India |

Click **Create**.

---

# Step 4: Create Azure Firewall

Navigate to:

```text
Azure Firewall
```

Click:

```text
Create
```

---

### Basics

| Property            | Value                             |
| ------------------- | --------------------------------- |
| Resource Group      | RG-Firewall-Demo                  |
| Name                | Hub-Firewall                      |
| Region              | Central India                     |
| Firewall SKU        | Standard                          |
| Firewall Management | Use Firewall Policy (Recommended) |

---

### Policy

Click:

```text
Create New Firewall Policy
```

Example:

```text
Hub-Firewall-Policy
```

---

### Public IP

Select:

```text
Firewall-PIP
```

---

### Virtual Network

Choose:

```text
Hub-VNet
```

Azure automatically detects:

```text
AzureFirewallSubnet
```

---

Click:

```text
Review + Create
```

Deployment usually takes **5–10 minutes**.

---

# Step 5: Verify

Open the Firewall resource.

You'll see:

```text
Private IP

10.0.0.4
```

and

```text
Public IP

20.x.x.x
```

The private IP will later be used as the **next hop** in User Defined Routes (UDRs).

---

# Azure Firewall Architecture

```text
Internet
      │
20.x.x.x
      │
Azure Firewall
Private IP
10.0.0.4
      │
Hub VNet
```

---

# Part 2: Deploy Azure Firewall using Azure CLI

## Step 1: Create Resource Group

```bash
az group create \
    --name RG-Firewall-Demo \
    --location centralindia
```

---

## Step 2: Create VNet

```bash
az network vnet create \
    --resource-group RG-Firewall-Demo \
    --name Hub-VNet \
    --address-prefix 10.0.0.0/16
```

---

## Step 3: Create AzureFirewallSubnet

```bash
az network vnet subnet create \
    --resource-group RG-Firewall-Demo \
    --vnet-name Hub-VNet \
    --name AzureFirewallSubnet \
    --address-prefixes 10.0.0.0/26
```

Notice:

The subnet name **must** be:

```text
AzureFirewallSubnet
```

Otherwise deployment fails.

---

## Step 4: Create Public IP

```bash
az network public-ip create \
    --resource-group RG-Firewall-Demo \
    --name Firewall-PIP \
    --sku Standard \
    --allocation-method Static
```

---

## Step 5: Create Firewall Policy

```bash
az network firewall policy create \
    --resource-group RG-Firewall-Demo \
    --name Hub-Firewall-Policy
```

---

## Step 6: Create Firewall

```bash
az network firewall create \
    --resource-group RG-Firewall-Demo \
    --name Hub-Firewall \
    --location centralindia \
    --firewall-policy Hub-Firewall-Policy
```

---

## Step 7: Associate Public IP

Retrieve the Public IP resource ID:

```bash
az network public-ip show \
    --resource-group RG-Firewall-Demo \
    --name Firewall-PIP \
    --query id \
    --output tsv
```

Store the output in a shell variable (Linux/macOS):

```bash
PIP_ID=$(az network public-ip show \
  --resource-group RG-Firewall-Demo \
  --name Firewall-PIP \
  --query id \
  --output tsv)
```

Or in PowerShell:

```powershell
$pipId = az network public-ip show `
  --resource-group RG-Firewall-Demo `
  --name Firewall-PIP `
  --query id `
  --output tsv
```

---

## Step 8: Configure Firewall IP

Retrieve the VNet ID:

```bash
az network vnet show \
    --resource-group RG-Firewall-Demo \
    --name Hub-VNet \
    --query id \
    --output tsv
```

Store it in a variable:

Linux/macOS:

```bash
VNET_ID=$(az network vnet show \
  --resource-group RG-Firewall-Demo \
  --name Hub-VNet \
  --query id \
  --output tsv)
```

PowerShell:

```powershell
$vnetId = az network vnet show `
  --resource-group RG-Firewall-Demo `
  --name Hub-VNet `
  --query id `
  --output tsv
```

Then configure the firewall IP:

Linux/macOS:

```bash
az network firewall ip-config create \
    --resource-group RG-Firewall-Demo \
    --firewall-name Hub-Firewall \
    --name FWConfig \
    --public-ip-address $PIP_ID \
    --vnet-name Hub-VNet
```

PowerShell:

```powershell
az network firewall ip-config create `
    --resource-group RG-Firewall-Demo `
    --firewall-name Hub-Firewall `
    --name FWConfig `
    --public-ip-address $pipId `
    --vnet-name Hub-VNet
```

---

## Step 9: Verify

```bash
az network firewall show \
    --resource-group RG-Firewall-Demo \
    --name Hub-Firewall
```

Look for:

```text
Private IP

10.0.0.4
```

and

```text
Public IP

20.x.x.x
```

---

# Resource Summary

After deployment, your Resource Group should contain:

```text
RG-Firewall-Demo
│
├── Hub-VNet
├── AzureFirewallSubnet
├── Firewall-PIP
├── Hub-Firewall
└── Hub-Firewall-Policy
```

---

# Next Lab

Once the firewall is deployed, the logical next step is to **route traffic through it**. We'll build a complete topology:

```text
Internet
      │
Azure Firewall
      │
Hub VNet
      │
User Defined Route (0.0.0.0/0)
      │
Spoke VNet
      │
Virtual Machine
```

Then we'll configure:

1. **SNAT** (VM → Internet)
2. **DNAT** (Internet → VM)
3. **Network Rules**
4. **Application Rules**

This sequence closely matches how Azure Firewall is implemented in production environments.
