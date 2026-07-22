This is a very common confusion. **Subnets and Availability Zones are completely different concepts.**

# 1. Does a Subnet Represent an Availability Zone?

**No.**

A subnet is an **IP boundary**, while an Availability Zone is a **physical datacenter boundary**.

---

## Example

```text
Central India Region
│
├── Zone1 (Datacenter)
├── Zone2 (Datacenter)
└── Zone3 (Datacenter)
```

Inside this region, you create one VNet:

```text
VNet : 10.0.0.0/16
│
├── Web Subnet : 10.0.1.0/24
├── App Subnet : 10.0.2.0/24
└── DB Subnet  : 10.0.3.0/24
```

The subnets are **not tied to any zone**.

---

# 2. Then How Are Availability Zones Selected?

Availability Zone is selected **when deploying the resource**, not when creating the subnet.

Example:

```bash
az vm create \
   --name webvm1 \
   --zone 1 \
   --vnet-name prod-vnet \
   --subnet web-subnet
```

Another VM:

```bash
az vm create \
   --name webvm2 \
   --zone 2 \
   --vnet-name prod-vnet \
   --subnet web-subnet
```

Both VMs are in:

```text
web-subnet (10.0.1.0/24)
```

But physically:

```text
webvm1 → Zone1
webvm2 → Zone2
```

---

# Architecture

```text
Central India Region

Zone1
   └── VM1
         ↓
      Web Subnet

Zone2
   └── VM2
         ↓
      Web Subnet
```

So a single subnet can contain VMs from multiple zones.

---

# 3. What If Availability Zones Are Not Supported?

Many regions do not support AZ.

Example:

```text
South India
```

You can still create:

```text
VNet
Subnets
VMs
NSGs
Load Balancers
```

Only zonal deployment options disappear.

Example:

```bash
az vm create
```

without:

```bash
--zone
```

Azure automatically deploys into the region.

---

# Example

```text
South India Region
│
├── VNet
│
├── Web Subnet
├── App Subnet
└── DB Subnet
```

Perfectly valid.

---

# 4. Can VMs in One Subnet Communicate With Another Subnet?

**Yes. By default they can.**

Azure automatically creates system routes.

Example:

```text
VNet : 10.0.0.0/16

Web Subnet : 10.0.1.0/24
App Subnet : 10.0.2.0/24
DB Subnet  : 10.0.3.0/24
```

Communication:

```text
VM(Web) → VM(App)
VM(App) → VM(DB)
VM(DB) → VM(Web)
```

All allowed by default.

---

# Why?

Azure creates a default route:

```text
10.0.0.0/16 → Virtual Network
```

---

# How to Block Communication?

Use NSGs.

Example:

```text
Deny Web → DB
Allow App → DB
```

---

# 5. Public and Private Subnets

This is another important concept.

Unlike AWS, **Azure does NOT have a subnet property called Public or Private.**

This often confuses AWS engineers.

---

# AWS

```text
Subnet Type:
✓ Public
✓ Private
```

---

# Azure

No such property exists.

A subnet becomes public or private depending upon the resources attached.

---

# Public Subnet

A subnet becomes effectively public if VMs have:

* Public IP
* Load Balancer Public Frontend
* NAT Rules

Example:

```text
Internet
     ↓
Public IP
     ↓
VM
     ↓
Web Subnet
```

---

# Private Subnet

No Public IP.

Example:

```text
Internet ❌

Web VM
     ↓
App VM
     ↓
DB VM
```

Access via:

* VPN
* Bastion
* Jump Server
* ExpressRoute

---

# Example Architecture

```text
VNet : 10.0.0.0/16

Web : 10.0.1.0/24
App : 10.0.2.0/24
DB  : 10.0.3.0/24
```

---

## Web Tier

```text
Public IP Attached
```

Effectively Public.

---

## App Tier

```text
No Public IP
```

Private.

---

## DB Tier

```text
No Public IP
```

Private.

---

# Typical Enterprise Design

```text
Internet
     ↓
Application Gateway
     ↓
Web Subnet

App Subnet

DB Subnet
```

---

# Public vs Private Example

```text
10.0.1.0/24  Web
     ↓
Public Load Balancer

10.0.2.0/24 App
     ↓
Private

10.0.3.0/24 DB
     ↓
Private
```

---

# How to Make a Subnet Private?

### Method 1

Do not assign Public IPs.

---

### Method 2

Add NSG:

```text
Deny Internet Inbound
```

---

### Method 3

Force traffic through Firewall.

```text
0.0.0.0/0
      ↓
Azure Firewall
```

using UDR.

---

# How Internet Access Works

Even without Public IPs, VMs may still get outbound internet connectivity.

For controlled outbound access use:

* NAT Gateway
* Azure Firewall

---

# Subnet Creation Example

---

## Create VNet

```bash
az network vnet create \
   -g demo-rg \
   -n prod-vnet \
   --address-prefix 10.0.0.0/16
```

---

## Create Web Subnet

```bash
az network vnet subnet create \
   -g demo-rg \
   --vnet-name prod-vnet \
   -n web-subnet \
   --address-prefixes 10.0.1.0/24
```

---

## App Subnet

```bash
az network vnet subnet create \
   -g demo-rg \
   --vnet-name prod-vnet \
   -n app-subnet \
   --address-prefixes 10.0.2.0/24
```

---

## DB Subnet

```bash
az network vnet subnet create \
   -g demo-rg \
   --vnet-name prod-vnet \
   -n db-subnet \
   --address-prefixes 10.0.3.0/24
```

---

# IP Planning Example

```text
10.0.0.0/16

GatewaySubnet        10.0.0.0/27
AzureBastionSubnet   10.0.0.32/26

Web                  10.0.1.0/24
App                  10.0.2.0/24
DB                   10.0.3.0/24

AKS                  10.0.4.0/22
Private Endpoints    10.0.8.0/24
```

---

# Service Endpoints

Service Endpoint extends VNet identity to Azure services.

Example:

```text
VM
 ↓
Storage Account
```

Traffic remains on Microsoft's backbone network.

Supported services:

* Storage
* SQL
* Key Vault
* Cosmos DB

---

## Enable Service Endpoint

```bash
az network vnet subnet update \
   -g demo-rg \
   --vnet-name prod-vnet \
   -n app-subnet \
   --service-endpoints Microsoft.Storage
```

---

# Delegated Subnets

Some Azure services require dedicated subnet control.

Examples:

* Azure Container Apps Environment
* App Service Environment
* Azure NetApp Files

Example:

```bash
az network vnet subnet update \
  -g demo-rg \
  --vnet-name prod-vnet \
  -n container-subnet \
  --delegations Microsoft.App/environments
```

---

# Summary

| Question                                       | Answer                   |
| ---------------------------------------------- | ------------------------ |
| Does subnet represent AZ?                      | No                       |
| Where is AZ selected?                          | On resource deployment   |
| Can same subnet contain VMs from multiple AZs? | Yes                      |
| Can subnets exist without AZ support?          | Yes                      |
| Can subnet-to-subnet communication happen?     | Yes, by default          |
| Does Azure have Public/Private subnet type?    | No                       |
| How make subnet private?                       | No Public IP + NSG + UDR |
| How make subnet public?                        | Attach Public IP/LB      |

---

## Example Interview Architecture

```text
Internet
      ↓
Application Gateway
      ↓
Web Subnet (Public)

App Subnet (Private)

DB Subnet (Private)
```

This is one of the most commonly used Azure enterprise network designs and is frequently asked in **AZ-104, AZ-700, and Solution Architect interviews**.
