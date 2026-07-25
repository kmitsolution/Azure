# Azure IP Addressing

IP addressing is one of the most important topics in Azure Networking because almost every Azure resource communicates using IP addresses.

In Azure, there are **two types of IP addresses**:

1. **Private IP Address**
2. **Public IP Address**

```text
                  Internet
                      │
               Public IP Address
                      │
                 Azure Firewall /
               Load Balancer / VM
                      │
               Private IP Address
                      │
              VNet → Subnet → VM
```

---

# 1. Private IP Address

A Private IP address is used for communication **inside a Virtual Network (VNet)**.

Examples:

```text
10.0.1.4
10.0.2.10
192.168.1.5
172.16.1.20
```

Private IPs are **not reachable from the Internet**.

---

## Example

```text
VNet : 10.0.0.0/16

Web VM : 10.0.1.4

App VM : 10.0.2.4

DB VM : 10.0.3.4
```

Communication:

```text
Web VM  → App VM
App VM  → DB VM
```

This communication happens using **Private IPs**.

---

# Characteristics of Private IPs

* Assigned to NICs
* Must belong to the subnet
* Used for internal communication
* Cannot be accessed directly from the Internet

---

# Dynamic Private IP

This is the default option.

Azure DHCP automatically assigns an available IP from the subnet.

Example:

```text
Subnet

10.0.1.0/24

↓

Azure assigns

10.0.1.4
```

You don't choose the IP.

---

## Advantages

* Easy management
* Automatic assignment
* No IP conflicts

---

## Disadvantage

If you delete and recreate the NIC, the IP may change.

---

# Static Private IP

You manually assign the IP.

Example:

```text
10.0.1.10
```

Azure reserves that IP for the NIC.

---

## When Should You Use Static Private IPs?

Examples:

* Domain Controllers
* DNS Servers
* Firewalls
* Load Balancers
* SQL Servers
* Appliances

---

## CLI Example

Create NIC with a static private IP:

```bash
az network nic create \
    --resource-group demo-rg \
    --name webnic \
    --vnet-name prod-vnet \
    --subnet web-subnet \
    --private-ip-address 10.0.1.10
```

---

# Dynamic vs Static Private IP

| Feature          | Dynamic | Static |
| ---------------- | ------- | ------ |
| Azure assigns IP | ✅       | ❌      |
| You choose IP    | ❌       | ✅      |
| Easy to manage   | ✅       | ❌      |
| Good for servers | ❌       | ✅      |

---

# 2. Public IP Address

A Public IP allows Azure resources to communicate with the Internet.

Example:

```text
Internet
      │
20.198.100.20
      │
VM
```

Without a Public IP, Internet users cannot directly reach your VM.

---

# Example

```text
Internet
     │
20.50.100.10
     │
Web VM
```

Users browse:

```
http://20.50.100.10
```

---

# Where Can Public IPs Be Attached?

Public IPs can be associated with:

* VM NIC
* Load Balancer
* Application Gateway
* Azure Firewall
* NAT Gateway
* Bastion

---

# Public IP Allocation Methods

Azure supports two allocation methods:

## 1. Static

The Public IP never changes.

Example:

```text
20.100.10.10
```

Even after VM restart, the IP remains the same.

---

### Use Cases

* Production Websites
* DNS Records
* VPN Gateway
* Firewalls

---

## 2. Dynamic

Azure assigns an available Public IP.

Example:

```text
20.100.10.20
```

Historically, dynamic Public IPs could change when the resource was deallocated and started again.

---

## Static vs Dynamic Public IP

| Feature     | Static | Dynamic    |
| ----------- | ------ | ---------- |
| IP changes  | No     | May change |
| Production  | ✅      | ❌          |
| Development | ✅      | ✅          |

> **Note:** With the Standard SKU (discussed below), Public IPs are static. Dynamic allocation was primarily associated with the older Basic SKU.

---

# Public IP SKU

Azure provides two SKUs:

1. Basic
2. Standard

---

# Basic SKU

Characteristics:

* Legacy SKU
* Supports Dynamic or Static allocation
* Open by default
* Limited feature set

Microsoft recommends using **Standard SKU** for new deployments.

---

# Standard SKU

Characteristics:

* Static Public IP only
* More secure (closed by default)
* Supports Availability Zones
* Better scalability and resiliency

---

## Basic vs Standard

| Feature                         | Basic | Standard |
| ------------------------------- | ----- | -------- |
| Static IP                       | ✅     | ✅        |
| Dynamic IP                      | ✅     | ❌        |
| Secure by Default               | ❌     | ✅        |
| Availability Zones              | ❌     | ✅        |
| Recommended for new deployments | ❌     | ✅        |

---

# Why "Secure by Default"?

Suppose you create a VM with a Standard Public IP.

```text
Internet
      │
Public IP
      │
VM
```

Even though the VM has a Public IP, **traffic is blocked until you explicitly allow it** (for example, with an NSG rule).

With the Basic SKU, inbound traffic behavior was more permissive.

---

# Public IP Lifecycle

```text
Create VM
      │
Attach Public IP
      │
Internet Connectivity
      │
Detach Public IP
      │
Internet Access Removed
```

The VM's Private IP remains unchanged.

---

# Practice: Attach Multiple Private IPs

### Create NIC

```bash
az network nic create \
    -g demo-rg \
    -n webnic \
    --vnet-name prod-vnet \
    --subnet web-subnet
```

---

### Add Second Private IP

```bash
az network nic ip-config create \
    -g demo-rg \
    --nic-name webnic \
    -n ipconfig2 \
    --private-ip-address 10.0.1.20
```

Now the NIC has:

```text
Primary IP    : 10.0.1.10

Secondary IP  : 10.0.1.20
```

---

# Attach Multiple Public IPs

Create a Public IP:

```bash
az network public-ip create \
    -g demo-rg \
    -n pip2 \
    --sku Standard
```

Associate it with the secondary IP configuration:

```bash
az network nic ip-config update \
    -g demo-rg \
    --nic-name webnic \
    -n ipconfig2 \
    --public-ip-address pip2
```

Result:

```text
Primary Private : 10.0.1.10
Primary Public  : 20.100.10.10

Secondary Private : 10.0.1.20
Secondary Public  : 20.100.10.11
```

---

# Enterprise Example

```text
Internet
      │
20.100.10.10
      │
Application Gateway
      │
Web VM

NIC
├── Primary IP : 10.0.1.4
└── Secondary IP : 10.0.1.5
```

The application can bind different services to different IP addresses if needed.

---

# Interview Questions

### Q1. Can a VM have multiple Private IPs?

**Yes.** A NIC can have multiple IP configurations.

---

### Q2. Can a VM have multiple Public IPs?

**Yes.** By associating Public IPs with different IP configurations on the NIC.

---

### Q3. Which Public IP SKU should be used for new deployments?

**Standard SKU**.

---

### Q4. Which IP is used for communication inside a VNet?

**Private IP**.

---

### Q5. Which IP allows Internet access?

**Public IP**.

---

# Summary

| Feature                    | Private IP | Public IP                                         |
| -------------------------- | ---------- | ------------------------------------------------- |
| Internet Accessible        | ❌          | ✅                                                 |
| Inside VNet Communication  | ✅          | ❌                                                 |
| Dynamic Allocation         | ✅          | Basic SKU only                                    |
| Static Allocation          | ✅          | ✅                                                 |
| Attached To                | NIC        | NIC, Load Balancer, Firewall, Application Gateway |
| Recommended for Production | Static     | Standard SKU                                      |

---

## Best Practices

* Use **Private IPs** for communication between Azure resources.
* Assign **Public IPs** only when external access is required.
* Prefer **Standard SKU Public IPs** for all new deployments.
* Use **Static IPs** for infrastructure components like VPN Gateways, firewalls, DNS servers, and production-facing services where the address must remain stable.
