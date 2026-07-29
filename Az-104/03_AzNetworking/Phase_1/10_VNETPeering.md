# Azure VNet Peering

VNet Peering allows **two Azure Virtual Networks to communicate privately using Microsoft's backbone network**.

There is **no VPN**, **no public Internet**, and **very low latency**.

Think of it as connecting two VNets with a high-speed private cable.

---

# Why Use VNet Peering?

Suppose you have two teams.

```text
Development Team

VNet-Dev
10.0.0.0/16
```

and

```text
Production Team

VNet-Prod
10.1.0.0/16
```

Without peering:

```text
VM1

❌ Cannot reach

VM2
```

After peering:

```text
VM1

──────────────►

VM2
```

Communication is over Microsoft's private backbone.

---

# Types of VNet Peering

## 1. Regional VNet Peering

Both VNets are in the same Azure Region.

Example:

```text
Central India

VNet-A

↓

VNet-B
```

---

## 2. Global VNet Peering

VNets are in different Azure Regions.

Example:

```text
Central India

VNet-A

──────────────►

East US

VNet-B
```

Traffic still uses Microsoft's backbone network.

---

# Architecture

```text
                Microsoft Backbone

VNet-A ------------------------ VNet-B

10.0.0.0/16                 10.1.0.0/16

VM1                          VM2
```

---

# Requirements

Before creating peering:

### Address spaces must NOT overlap

Allowed

```text
VNet-A

10.0.0.0/16

VNet-B

10.1.0.0/16
```

Not allowed

```text
10.0.0.0/16

10.0.1.0/24
```

because they overlap.

---

# Hands-on Lab (Azure Portal)

## Goal

Create:

```text
VNet1

10.0.0.0/16

↓

Peering

↓

VNet2

10.1.0.0/16
```

Deploy one VM in each VNet and verify connectivity.

---

# Step 1: Create Resource Group

```text
RG-Peering
```

Region:

```text
Central India
```

---

# Step 2: Create VNet-A

Navigate:

```text
Virtual Networks

↓

Create
```

Configuration:

| Property      | Value       |
| ------------- | ----------- |
| Name          | VNet-A      |
| Address Space | 10.0.0.0/16 |

Subnet

```text
10.0.1.0/24
```

---

# Step 3: Create VNet-B

Create another VNet.

| Property      | Value       |
| ------------- | ----------- |
| Name          | VNet-B      |
| Address Space | 10.1.0.0/16 |

Subnet

```text
10.1.1.0/24
```

---

# Step 4: Create VM1

Inside:

```text
VNet-A
```

VM Name

```text
VM-A
```

Private IP

```text
10.0.1.4
```

No Public IP required for testing (or use Azure Bastion/Run Command).

---

# Step 5: Create VM2

Inside

```text
VNet-B
```

VM Name

```text
VM-B
```

Private IP

```text
10.1.1.4
```

---

# Step 6: Create Peering

Open:

```text
VNet-A

↓

Peerings

↓

Add
```

Configure:

| Property     | Value  |
| ------------ | ------ |
| Peering Name | A-To-B |
| Remote VNet  | VNet-B |

Enable:

```text
Allow Virtual Network Access

✔
```

Leave Gateway Transit and Forwarded Traffic disabled for this basic lab.

Click **Add**.

---

# Step 7: Create Reverse Peering

Now go to:

```text
VNet-B

↓

Peerings

↓

Add
```

Create:

```text
B-To-A
```

Remote Network:

```text
VNet-A
```

Enable:

```text
Allow Virtual Network Access
```

Click **Add**.

---

# Final Architecture

```text
VNet-A

10.0.0.0/16

VM-A

10.0.1.4

──────────────►

VNet-B

10.1.0.0/16

VM-B

10.1.1.4
```

---

# Step 8: Test

From VM-A

Linux

```bash
ping 10.1.1.4
```

Windows

```cmd
ping 10.1.1.4
```

Expected

```text
Reply from 10.1.1.4
```

Success.

---

# Azure CLI

---

## Create Resource Group

```bash
az group create \
    --name RG-Peering \
    --location centralindia
```

---

## Create VNet-A

```bash
az network vnet create \
    -g RG-Peering \
    -n VNet-A \
    --address-prefix 10.0.0.0/16 \
    --subnet-name WebSubnet \
    --subnet-prefix 10.0.1.0/24
```

---

## Create VNet-B

```bash
az network vnet create \
    -g RG-Peering \
    -n VNet-B \
    --address-prefix 10.1.0.0/16 \
    --subnet-name AppSubnet \
    --subnet-prefix 10.1.1.0/24
```

---

## Create Peering

### A → B

```bash
az network vnet peering create \
    -g RG-Peering \
    --vnet-name VNet-A \
    -n A-To-B \
    --remote-vnet VNet-B \
    --allow-vnet-access
```

---

### B → A

```bash
az network vnet peering create \
    -g RG-Peering \
    --vnet-name VNet-B \
    -n B-To-A \
    --remote-vnet VNet-A \
    --allow-vnet-access
```

---

## Verify

```bash
az network vnet peering list \
    -g RG-Peering \
    --vnet-name VNet-A
```

---

# Communication

After peering

```text
VM-A

↓

VM-B

✔
```

---

# What Is NOT Allowed?

Suppose

```text
VNet-A

10.0.0.0/16

VNet-B

10.0.0.0/16
```

Peering fails.

Azure displays

```text
Address space overlaps.
```

---

# Can NSGs Block Peering?

Yes.

Suppose peering exists.

```text
VM1

↓

VM2
```

But NSG

```text
Deny ICMP
```

Result

```text
Ping

Fails
```

Peering only creates connectivity. **NSGs still control whether the traffic is allowed.**

---

# Gateway Transit

Suppose

```text
On-Prem

↓

VPN Gateway

↓

Hub VNet

↓

Spoke1

↓

Spoke2
```

Instead of deploying VPN Gateways in every spoke, the spokes can use the Hub's VPN Gateway.

This is called:

```text
Gateway Transit
```

Used in Hub-Spoke architectures.

---

# Hub-Spoke Example

```text
           On-Prem

              │

         VPN Gateway

              │

         Hub VNet

      ┌─────┴─────┐

  Spoke1      Spoke2

  Dev         Prod
```

---

# Advantages

* High bandwidth
* Low latency
* Microsoft backbone
* No VPN required
* Easy management

---

# Limitations

### No Overlapping IPs

Mandatory.

---

### Not Transitive

This is a very common interview question.

Example

```text
VNet-A

↓

Peered

↓

VNet-B

↓

Peered

↓

VNet-C
```

Can A talk to C?

**No.**

You need another peering.

```text
VNet-A

────────────►

VNet-C
```

or use Azure Virtual WAN / Hub-and-Spoke with proper routing.

---

# Interview Questions

## Q1. What is VNet Peering?

A private connection between two Azure VNets over Microsoft's backbone network.

---

## Q2. Does VNet Peering use the Internet?

**No.**

It uses Microsoft's private backbone.

---

## Q3. Can peering work across regions?

**Yes.**

That's called **Global VNet Peering**.

---

## Q4. Can VNets with overlapping address spaces be peered?

**No.**

---

## Q5. Is VNet Peering encrypted?

Traffic stays on Microsoft's private backbone. If your application requires encryption, you should use protocols like **TLS**, **IPsec**, or application-level encryption. Don't assume VNet Peering itself provides end-to-end encryption for all traffic.

---

## Q6. Is VNet Peering transitive?

**No.**

---

## Q7. Can NSGs affect peered VNets?

**Yes.**

NSGs are still enforced on subnet and NIC traffic.

---

## Q8. Can a VM have two VNets instead of peering?

**No.**

A NIC belongs to only one VNet.

---

## Q9. Can you peer VNets in different subscriptions?

**Yes.**

As long as you have the required permissions and there are no overlapping address spaces.

---

## Q10. What is the difference between VNet Peering and VPN Gateway?

| VNet Peering                | VPN Gateway                          |
| --------------------------- | ------------------------------------ |
| Uses Microsoft backbone     | Uses IPsec VPN tunnels               |
| Very low latency            | Higher latency due to tunneling      |
| No VPN appliance required   | VPN Gateway required                 |
| Azure-to-Azure connectivity | Azure-to-Azure or On-prem-to-Azure   |
| High bandwidth              | Lower than peering in most scenarios |

---

# Best Practice

For enterprise environments, a common design is:

```text
              Hub VNet
                  │
     ┌────────────┼────────────┐
     │            │            │
  Dev VNet     Test VNet    Prod VNet
```

Each spoke VNet is peered with the Hub VNet, allowing centralized services like Azure Firewall, VPN Gateway, or DNS to be shared across all environments. This is one of the most widely used Azure network architectures.
