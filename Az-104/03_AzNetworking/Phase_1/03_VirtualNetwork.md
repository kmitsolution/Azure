# Azure Virtual Networks (VNet)

A Virtual Network (VNet) is the fundamental networking building block in Azure.

Think of a VNet as your **own private network inside Azure**, similar to a traditional on-premises network in a datacenter.

---

# 1. What is a VNet?

A VNet allows Azure resources to communicate securely with:

* Other Azure resources
* The Internet
* On-premises networks
* Other VNets

---

## Traditional Datacenter

```text
Physical Network
     ↓
Switches
     ↓
Servers
```

## Azure

```text
Virtual Network (VNet)
        ↓
Subnets
        ↓
VMs / AKS / App Services / Databases
```

---

## Example Architecture

```text
VNet : 10.0.0.0/16

├── Web Subnet : 10.0.1.0/24
├── App Subnet : 10.0.2.0/24
└── DB Subnet  : 10.0.3.0/24
```

---

## Characteristics of VNet

* Logical isolation
* Private IP addressing
* Routing support
* DNS integration
* Security integration (NSG)
* Hybrid connectivity support

---

## Supported Communication

```text
VM ↔ VM
VM ↔ Internet
VM ↔ On-Prem
VNet ↔ VNet
AKS ↔ Storage
Private Endpoint ↔ SQL
```

---

# Create VNet using CLI

```bash
az network vnet create \
   --resource-group demo-rg \
   --name prod-vnet \
   --address-prefix 10.0.0.0/16
```

---

# 2. Address Spaces

Every VNet requires one or more address spaces.

Address space defines the IP range available inside the VNet.

Example:

```text
10.0.0.0/16
```

means:

```text
Network Portion : 16 bits
Host Portion    : 16 bits
```

Available IPs:

```text
65536 addresses
```

---

## Example

```text
VNet : 10.0.0.0/16

Possible Subnets:

10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
10.0.4.0/24
```

---

# Recommended Practice

For enterprise environments:

| Environment | Address Space |
| ----------- | ------------- |
| Dev         | 10.10.0.0/16  |
| Test        | 10.20.0.0/16  |
| Prod        | 10.30.0.0/16  |

This avoids overlap later.

---

# Interview Question

### Can VNet have multiple address spaces?

Yes.

A VNet can have multiple non-overlapping address spaces.

---

# 3. Multiple Address Spaces

Sometimes one address space becomes insufficient.

You can add another address range.

Example:

```text
VNet

Address Space 1:
10.0.0.0/16

Address Space 2:
192.168.0.0/16
```

---

## Why Multiple Address Spaces?

Examples:

### Scenario 1

Application expansion.

### Scenario 2

After VNet peering, additional address requirements arise.

### Scenario 3

Migration from on-premises.

---

## CLI Example

```bash
az network vnet update \
   --resource-group demo-rg \
   --name prod-vnet \
   --address-prefixes 10.0.0.0/16 192.168.0.0/16
```

---

# Important Rule

Address spaces inside a VNet:

✅ Must not overlap.

Example:

```text
10.0.0.0/16
192.168.0.0/16
```

Allowed.

---

❌ Not allowed:

```text
10.0.0.0/16
10.0.1.0/24
```

because second range already belongs to first range.

---

# 4. CIDR Planning (Extremely Important)

Poor IP planning becomes a major problem in enterprises.

---

# Example Bad Design

```text
Dev VNet      10.0.0.0/16
Test VNet     10.0.0.0/16
Prod VNet     10.0.0.0/16
```

VNet peering later becomes impossible due to overlapping ranges.

---

# Good Design

```text
Hub        : 10.0.0.0/16
Dev        : 10.1.0.0/16
Test       : 10.2.0.0/16
Prod       : 10.3.0.0/16
DR         : 10.4.0.0/16
```

---

# Large Enterprise Example

```text
Global Hub      : 10.0.0.0/12
India           : 10.16.0.0/12
US              : 10.32.0.0/12
Europe           :10.48.0.0/12
```

---

# Hub-Spoke Planning Example

```text
Hub        : 10.0.0.0/16
Shared     : 10.1.0.0/16
Dev        : 10.2.0.0/16
Test       : 10.3.0.0/16
Prod       : 10.4.0.0/16
```

---

# Subnet Planning

```text
VNet : 10.1.0.0/16

Web      : 10.1.1.0/24
App      : 10.1.2.0/24
DB       : 10.1.3.0/24
AKS      : 10.1.4.0/22
Gateway  : 10.1.10.0/27
Bastion  : 10.1.11.0/26
```

---

# Important Azure Reserved Subnets

| Service       | Required Subnet     |
| ------------- | ------------------- |
| VPN Gateway   | GatewaySubnet       |
| Azure Bastion | AzureBastionSubnet  |
| Firewall      | AzureFirewallSubnet |

Names are case-sensitive.

---

# CIDR Recommendations

| Resource      | Recommended   |
| ------------- | ------------- |
| Web Tier      | /24           |
| AKS           | /22 or larger |
| GatewaySubnet | /27 or larger |
| Bastion       | /26           |
| Small Apps    | /24           |

---

# 5. Reserved Azure IP Addresses

Azure reserves **5 IP addresses in every subnet**.

Example:

Subnet:

```text
10.0.1.0/24
```

Total IPs:

```text
256
```

Usable:

```text
251
```

---

# Reserved Addresses

```text
10.0.1.0
10.0.1.1
10.0.1.2
10.0.1.3
10.0.1.255
```

---

## Why Reserved?

| IP   | Purpose                 |
| ---- | ----------------------- |
| .0   | Network ID              |
| .1   | Azure Gateway           |
| .2   | Azure DNS               |
| .3   | Future Azure Use        |
| Last | Broadcast compatibility |

---

## Example

Subnet:

```text
10.0.1.0/29
```

Total:

```text
8 IPs
```

Azure reserves:

```text
5
```

Remaining:

```text
3 usable addresses
```

This is why small subnets often cause deployment failures.

---

# Interview Question

### Why can't I deploy many VMs into a /29 subnet?

Because Azure reserves five IP addresses.

---

# 6. VNet Limits

---

# Address Space Limit

A VNet can contain multiple address spaces.

---

# Subnet Limits

Azure supports a large number of subnets per VNet (limits may vary by subscription and service evolution).

---

# Peering Limits

VNets can be peered with many other VNets, but always verify current limits for large-scale architectures.

---

# Private Endpoint Limits

Private Endpoints consume IP addresses from subnets.

Always allocate larger subnets if many private endpoints are expected.

---

# Service Endpoint Limits

Multiple service endpoints can coexist inside a VNet.

---

# Design Recommendation

Avoid:

```text
10.0.1.0/29
10.0.2.0/29
```

Prefer:

```text
10.0.1.0/24
10.0.2.0/24
```

because cloud resources grow over time.

---

# Enterprise Architecture Example

```text
Hub VNet
10.0.0.0/16

Spoke-Dev
10.1.0.0/16

Spoke-Test
10.2.0.0/16

Spoke-Prod
10.3.0.0/16
```

---

# Real Interview Questions

### Q1. Can a VM belong to multiple VNets?

No.

A NIC belongs to only one VNet.

---

### Q2. Can subnets overlap?

No.

---

### Q3. Can VNets overlap?

Yes.

But overlapping VNets cannot be peered.

---

### Q4. Can address spaces be added later?

Yes.

Provided they do not overlap.

---

# Hands-On Labs

---

## Lab 1

Create VNet:

```bash
az network vnet create \
    -g demo-rg \
    -n prod-vnet \
    --address-prefix 10.0.0.0/16
```

---

## Lab 2

Create Subnets:

```bash
az network vnet subnet create \
   -g demo-rg \
   --vnet-name prod-vnet \
   -n web-subnet \
   --address-prefix 10.0.1.0/24
```

---

## Lab 3

Create VM in subnet:

```bash
az vm create \
   -g demo-rg \
   -n webvm1 \
   --vnet-name prod-vnet \
   --subnet web-subnet
```

---

## Lab 4

Add second address space:

```bash
az network vnet update \
   -g demo-rg \
   -n prod-vnet \
   --address-prefixes 10.0.0.0/16 192.168.0.0/16
```

---

# Architecture Diagram

```text
Internet
      ↓
Public IP
      ↓
Load Balancer
      ↓
VNet (10.0.0.0/16)

 ├── Web     (10.0.1.0/24)
 ├── App     (10.0.2.0/24)
 ├── DB      (10.0.3.0/24)
 └── Gateway (10.0.10.0/27)
```

Yes, **VNets are region-specific resources in Azure.**

## Example

If you create a VNet in **Central India**, that VNet exists only in the **Central India** region.

```text
Region: Central India
VNet: prod-vnet
Address Space: 10.0.0.0/16
```

You cannot directly move this VNet to another region like East US.

---

# Important Point

All resources connected to that VNet must generally be deployed in the same region.

For example:

```text
Central India
│
├── VNet
├── VM1
├── VM2
└── Load Balancer
```

This works because all resources are in the same region.

---

# Can resources from another region use this VNet?

No, because a VNet does not span multiple regions.

For example:

```text
Central India VNet
        ↓
East US VM
```

This is **not possible directly**.

Instead, you need:

* VNet Peering (Global VNet Peering)
* VPN Gateway
* ExpressRoute

---

# Example Architecture

```text
Central India
   │
   └── VNet-A (10.0.0.0/16)

Global Peering
         ⇅

East US
   │
   └── VNet-B (10.1.0.0/16)
```

---

# Can I create subnets in different regions inside one VNet?

No.

```text
VNet
 ├── Subnet1 → Central India
 └── Subnet2 → East US
```

This architecture is impossible.

A VNet and all its subnets belong to the same region.

---

# Are there any exceptions?

Some Azure services are **global**, but VNets are not.

Examples of global services:

* Azure DNS
* Azure Front Door
* Traffic Manager
* Azure Entra ID (Azure AD)

Examples of regional services:

* VNet
* VM
* Application Gateway
* Load Balancer
* Azure Firewall

---

# Interview Question

### Q: Can one VNet span multiple Azure regions?

**Answer: No.**

VNets are regional resources.

To connect networks across regions, use:

```text
Region1 VNet  ←→  Region2 VNet
```

through:

1. Global VNet Peering
2. VNet-to-VNet VPN
3. ExpressRoute Global Reach

---

# Real Production Example

```text
Central India
    │
    └── Hub-VNet (10.0.0.0/16)

Global VNet Peering
          ⇅

South India
    │
    └── DR-VNet (10.1.0.0/16)
```

This is a common Disaster Recovery architecture.

---

# One More Important Concept

Although VNets are regional, **their address spaces do not have to match the region**.

This is perfectly valid:

```text
Region: Central India
VNet Address Space: 192.168.100.0/16
```

The IP range and the Azure region are completely independent concepts.

---

## Quick Summary

| Question                                    | Answer                                       |
| ------------------------------------------- | -------------------------------------------- |
| Is VNet region specific?                    | Yes                                          |
| Can one VNet span multiple regions?         | No                                           |
| Can subnets be in different regions?        | No                                           |
| Can VNets in different regions communicate? | Yes, using Global VNet Peering/VPN/ER        |
| Can VNet be moved to another region?        | Not directly; recreate and migrate resources |

This question is asked very frequently in **AZ-104, AZ-700, and Azure Solution Architect interviews**.

