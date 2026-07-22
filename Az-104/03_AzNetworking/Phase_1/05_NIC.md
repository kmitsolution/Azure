# Azure Network Interfaces (NIC)

A Network Interface (NIC) is a virtual network card that enables an Azure VM to communicate with networks.

Think of it exactly like the physical network card in your laptop.

```text
Laptop
   ↓
Physical NIC
```

In Azure:

```text
VM
 ↓
Virtual NIC (vNIC)
 ↓
Subnet
 ↓
VNet
```

---

# Architecture

```text
VM
 ↓
NIC
 ↓
Subnet
 ↓
VNet
```

A VM cannot communicate without a NIC.

---

# Important Points

A NIC contains:

* Private IP addresses
* Public IP association
* NSGs
* DNS settings
* IP forwarding settings

---

# Example

```text
VNet : 10.0.0.0/16
      ↓
Web Subnet : 10.0.1.0/24
      ↓
NIC : webvm1-nic
      ↓
VM : webvm1
```

---

# 1. Primary IP Configuration

Every NIC must have one **Primary Private IP**.

Example:

```text
NIC
 ↓
Primary IP : 10.0.1.4
```

This IP is used for:

* Default outbound traffic
* DNS registration
* Default communication

---

## Example

```text
VM
 ↓
NIC
 ↓
10.0.1.4 (Primary)
```

---

## CLI Example

Create NIC:

```bash
az network nic create \
   -g demo-rg \
   -n webnic \
   --vnet-name prod-vnet \
   --subnet web-subnet
```

Azure automatically assigns a primary IP.

---

## Static Primary IP

```bash
az network nic ip-config update \
   -g demo-rg \
   --nic-name webnic \
   -n ipconfig1 \
   --private-ip-address 10.0.1.10
```

---

# Dynamic vs Static Private IP

### Dynamic

Assigned by Azure DHCP.

Example:

```text
10.0.1.4
```

May change after NIC recreation.

---

### Static

You choose:

```text
10.0.1.10
```

Usually recommended for:

* Domain Controllers
* DNS Servers
* Firewalls

---

# 2. Secondary IP Addresses

A NIC can have multiple private IP addresses.

Example:

```text
Primary IP     : 10.0.1.4
Secondary IP1  : 10.0.1.5
Secondary IP2  : 10.0.1.6
```

---

# Why Use Secondary IPs?

Examples:

### Multiple Websites

```text
www.site1.com → 10.0.1.4
www.site2.com → 10.0.1.5
```

---

### Firewall Appliances

Many network virtual appliances use multiple IPs.

---

### SQL Clusters

---

# Add Secondary IP

```bash
az network nic ip-config create \
   -g demo-rg \
   --nic-name webnic \
   -n secondary1 \
   --private-ip-address 10.0.1.5
```

---

# Architecture

```text
VM
 ↓
NIC
 ├── 10.0.1.4
 ├── 10.0.1.5
 └── 10.0.1.6
```

---

# Public IP Association

Each IP configuration can also have a Public IP.

Example:

```text
10.0.1.4 → 20.10.10.10
10.0.1.5 → 20.10.10.11
```

---

# Interview Question

### Can one NIC have multiple IP addresses?

Yes.

Both private and public IP configurations are supported.

---

# 3. Multiple NICs

A VM can have more than one NIC.

---

## Example

```text
VM
 ├── NIC1
 └── NIC2
```

---

# Why Multiple NICs?

### Firewall Appliances

```text
Internet
    ↓
NIC1 (External)

Firewall VM

NIC2 (Internal)
```

---

### Multi-homed Servers

### Security Separation

### Routing Appliances

---

# Example Architecture

```text
Web VM
 ├── NIC1 → Frontend Subnet
 └── NIC2 → Backend Subnet
```

---

# Azure Example

```text
VNet
│
├── Frontend : 10.0.1.0/24
└── Backend  : 10.0.2.0/24
```

VM:

```text
NIC1 → Frontend
NIC2 → Backend
```

---

# Important Rule

Multiple NICs must belong to:

✅ Same VNet

❌ Different VNets are not allowed.

---

# Example

Allowed:

```text
NIC1 → Web Subnet
NIC2 → App Subnet
```

because both are in same VNet.

---

Not allowed:

```text
NIC1 → VNet1
NIC2 → VNet2
```

---

# Can Multiple NICs Be in Different Subnets?

Yes.

Example:

```text
NIC1 → Web Subnet
NIC2 → DB Subnet
```

---

# Example CLI

Create second NIC:

```bash
az network nic create \
   -g demo-rg \
   -n backendnic \
   --vnet-name prod-vnet \
   --subnet app-subnet
```

Attach to VM:

```bash
az vm nic add \
   -g demo-rg \
   --vm-name myvm \
   --nics backendnic
```

---

# Primary NIC

If multiple NICs exist:

One NIC becomes the Primary NIC.

It handles:

* Default routing
* Default gateway
* Internet traffic

---

# Example

```text
VM
├── NIC1 (Primary)
└── NIC2
```

---

# Interview Question

### Can VM have multiple primary NICs?

No.

Only one primary NIC is allowed.

---

# 4. Accelerated Networking

This is a very important performance feature.

---

## What is Accelerated Networking?

Azure bypasses part of the virtual switch processing.

Uses:

```text
SR-IOV
(Single Root I/O Virtualization)
```

---

# Normal Networking

```text
VM
 ↓
Hypervisor
 ↓
Virtual Switch
 ↓
Network
```

---

# Accelerated Networking

```text
VM
 ↓
Hardware NIC
 ↓
Network
```

---

# Benefits

* Lower latency
* Higher packets per second
* Lower CPU utilization
* Better throughput

---

# Example

Without Accelerated Networking:

```text
Latency : Higher
CPU      : Higher
```

With Accelerated Networking:

```text
Latency : Lower
CPU      : Lower
```

---

# Common Use Cases

### SQL Servers

### Kubernetes Nodes

### High Traffic APIs

### Firewalls

### Financial Applications

---

# Enable During NIC Creation

```bash
az network nic create \
   -g demo-rg \
   -n fastnic \
   --vnet-name prod-vnet \
   --subnet web-subnet \
   --accelerated-networking true
```

---

# Portal Option

```text
VM Creation
   ↓
Networking
   ↓
Enable Accelerated Networking
```

---

# Important Limitation

Not all VM sizes support it.

Examples supporting it:

* Dv2
* Dv3
* Dv5
* Ev3
* Fsv2

---

# Verify

```bash
az network nic show \
   -g demo-rg \
   -n fastnic
```

---

# NIC Security

NSGs can be attached:

### At NIC level

```text
NIC → NSG
```

### At Subnet level

```text
Subnet → NSG
```

---

# Processing Order

Traffic is evaluated against both subnet and NIC NSGs.

---

# Example Architecture

```text
Internet
      ↓
Public IP
      ↓
NIC
      ↓
VM
      ↓
Web Subnet
      ↓
VNet
```

---

# Complete Enterprise Example

```text
Hub VNet
│
├── Firewall VM
│     ├── NIC1 → External
│     └── NIC2 → Internal
│
├── Web VM
│     └── NIC1 → Web Subnet
│
└── SQL VM
      ├── NIC1 → App Subnet
      └── NIC2 → Backup Subnet
```

---

# Interview Questions

### Q1. Is NIC regional?

Yes.

NIC must be in the same region as the VM and VNet.

---

### Q2. Can NIC move between VNets?

No.

NIC is tied to a specific VNet.

---

### Q3. Can VM communicate without NIC?

No.

---

### Q4. Can multiple VMs share one NIC?

No.

One NIC belongs to one VM.

---

### Q5. Can NIC have multiple private IPs?

Yes.

---

### Q6. Can NIC belong to multiple VNets?

No.

---

# Summary

| Feature                   | Supported |
| ------------------------- | --------- |
| Multiple Private IPs      | Yes       |
| Multiple Public IPs       | Yes       |
| Multiple NICs per VM      | Yes       |
| Multiple VNets per VM     | No        |
| Different Subnets per NIC | Yes       |
| Accelerated Networking    | Yes       |
| Multiple Primary NICs     | No        |

---

# Typical Three-Tier Architecture

```text
Internet
     ↓
Public IP
     ↓
Web VM
     ↓
Web Subnet
     ↓
App Subnet
     ↓
DB Subnet
```

Each VM uses one or more NICs to participate in communication inside the VNet. Understanding NICs is extremely important because NSGs, Public IPs, Load Balancers, Application Gateway backend pools, and Private Endpoints all ultimately connect through NIC concepts.
