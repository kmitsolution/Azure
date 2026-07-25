# Azure Network Security Groups (NSG)

A **Network Security Group (NSG)** is Azure's virtual firewall.

It is used to **allow or deny network traffic** to Azure resources based on rules.

Think of it like a security guard at the entrance of your office.

```text
Visitor Arrives
       │
       ▼
Security Guard (NSG)
       │
 ┌─────┴─────┐
 │           │
Allow     Deny
```

An NSG doesn't inspect application content; it filters traffic based on network information such as source, destination, protocol, and port.

---

# Where Can NSGs Be Applied?

An NSG can be associated with:

1. **Subnet**
2. **Network Interface (NIC)**

```text
VNet
│
├── Web Subnet
│      │
│      └── NSG
│
├── App Subnet
│      │
│      └── VM
│            │
│            └── NIC
│                 │
│                 └── NSG
```

You can attach an NSG to either location or even to both.

---

# Why Use NSGs?

Suppose you have:

```text
Internet
     │
     ▼
Web Server
```

Without an NSG:

* Anyone can attempt to connect.
* All ports are potentially reachable if exposed.

With an NSG:

```text
Allow HTTP (80)

Allow HTTPS (443)

Allow RDP (3389) only from Admin IP

Deny Everything Else
```

Now only the required traffic is allowed.

---

# How NSG Works

Every packet entering or leaving a VM is evaluated against NSG rules.

Example:

```text
Internet
      │
      ▼
NSG
      │
      ▼
VM
```

Azure checks:

* Source IP
* Destination IP
* Port
* Protocol
* Direction

---

# NSG Rule Components

Each rule contains:

| Property         | Description                               |
| ---------------- | ----------------------------------------- |
| Priority         | Rule evaluation order                     |
| Source           | Source IP, subnet, service tag, or ASG    |
| Source Port      | Usually `*`                               |
| Destination      | IP, subnet, service tag, or ASG           |
| Destination Port | 80, 443, 22, etc.                         |
| Protocol         | TCP, UDP, ICMP (where applicable), or Any |
| Action           | Allow or Deny                             |
| Direction        | Inbound or Outbound                       |

---

# Example Rule

Allow HTTP:

```text
Priority : 100

Source : *

Destination : *

Protocol : TCP

Port : 80

Action : Allow
```

---

# Rule Priority

Priority determines which rule is processed first.

Lower number = Higher priority.

Example:

```text
100 Allow HTTP

200 Allow HTTPS

300 Deny All
```

Traffic to port 80:

Rule 100 matches.

Azure stops processing.

---

# Example

```text
100 Allow SSH

200 Deny All
```

SSH:

Allowed.

HTTP:

Denied.

---

# Interview Question

Which rule executes first?

```text
Priority 100

Priority 300
```

**Answer: Priority 100**, because the lowest priority number has the highest precedence.

---

# Inbound Rules

Inbound rules control traffic entering the VM.

```text
Internet
      │
      ▼
VM
```

Example:

Allow:

* HTTP
* HTTPS
* SSH

Deny:

Everything else.

---

# Outbound Rules

Outbound rules control traffic leaving the VM.

Example:

```text
VM
 │
 ▼
Internet
```

You can:

Allow:

* DNS
* HTTPS

Deny:

* Internet access
* Certain destinations

---

# Default NSG Rules

Azure automatically creates default rules.

---

## Default Inbound Rules

| Priority | Rule                          | Action |
| -------- | ----------------------------- | ------ |
| 65000    | AllowVNetInBound              | Allow  |
| 65001    | AllowAzureLoadBalancerInBound | Allow  |
| 65500    | DenyAllInbound                | Deny   |

---

## Default Outbound Rules

| Priority | Rule                  | Action |
| -------- | --------------------- | ------ |
| 65000    | AllowVNetOutBound     | Allow  |
| 65001    | AllowInternetOutBound | Allow  |
| 65500    | DenyAllOutBound       | Deny   |

---

# Important Point

You cannot delete the default rules.

You can override them with higher-priority custom rules.

Example:

```text
100 Deny Internet

65001 Allow Internet
```

Traffic is denied because rule 100 is evaluated first.

---

# NSG Processing

## Subnet NSG Only

```text
Internet
     │
Subnet NSG
     │
VM
```

Traffic is filtered once.

---

## NIC NSG Only

```text
Internet
      │
VM
      │
NIC NSG
```

Traffic is filtered once.

---

## Both Applied

```text
Internet
      │
Subnet NSG
      │
NIC NSG
      │
VM
```

Traffic must be allowed by **both**.

---

# Evaluation Order

## Inbound

```text
Internet

↓

Subnet NSG

↓

NIC NSG

↓

VM
```

---

## Outbound

```text
VM

↓

NIC NSG

↓

Subnet NSG

↓

Internet
```

This is a very common interview question.

---

# Service Tags

Instead of using IP addresses, Azure provides Service Tags.

Examples:

* Internet
* VirtualNetwork
* AzureLoadBalancer
* Storage
* SQL
* KeyVault

Example:

```text
Source

Storage
```

instead of thousands of IP addresses.

---

# Application Security Groups (ASG)

Instead of IP addresses, you can group VMs.

Example:

```text
Web ASG

App ASG

DB ASG
```

Rule:

```text
Web ASG

↓

App ASG

Allow 443
```

This makes rules easier to manage.

---

# Example Three-Tier Architecture

```text
Internet
      │
Application Gateway
      │
Web Subnet
      │
App Subnet
      │
DB Subnet
```

### NSG Rules

## Web

Allow:

* 80
* 443

---

## App

Allow:

* 443 from Web subnet

---

## Database

Allow:

* 1433 from App subnet

Deny:

Everything else.

---

# Create NSG

```bash
az network nsg create \
   -g demo-rg \
   -n web-nsg
```

---

# Add Rule

Allow HTTP:

```bash
az network nsg rule create \
   -g demo-rg \
   --nsg-name web-nsg \
   -n AllowHTTP \
   --priority 100 \
   --direction Inbound \
   --access Allow \
   --protocol Tcp \
   --destination-port-ranges 80
```

---

# Associate NSG with Subnet

```bash
az network vnet subnet update \
   -g demo-rg \
   --vnet-name prod-vnet \
   -n web-subnet \
   --network-security-group web-nsg
```

---

# Associate NSG with NIC

```bash
az network nic update \
   -g demo-rg \
   -n webnic \
   --network-security-group web-nsg
```

---

# Best Practices

### Web Tier

Allow:

* 80
* 443

Restrict:

* SSH/RDP to trusted administrator IP addresses only.

---

### App Tier

Allow:

* Traffic only from the Web subnet.

No Public IP.

---

### Database Tier

Allow:

* SQL only from the App subnet.

No Internet access.

---

# Common Mistakes

### 1. Using Low Priority Incorrectly

```text
100 Deny All

200 Allow HTTP
```

HTTP will never work because "Deny All" matches first.

---

### 2. Forgetting NIC NSG

The subnet NSG allows traffic.

The NIC NSG denies it.

Result:

Connection fails.

---

### 3. Opening RDP/SSH to Everyone

```text
Source: *
Port: 3389
```

This is insecure.

Instead:

```text
Source:
203.0.113.10/32
```

(Replace with your organization's public IP.)

---

# Real Enterprise Example

```text
Internet
      │
Application Gateway
      │
Web NSG
      │
Web VM
      │
App NSG
      │
App VM
      │
DB NSG
      │
SQL VM
```

Each tier only communicates with the next tier.

---

# Frequently Asked Interview Questions

### Q1. Does an NSG work at Layer 7?

**No.**

NSGs operate primarily at **OSI Layer 3 (Network)** and **Layer 4 (Transport)** by filtering based on IP addresses, protocols, and ports.

---

### Q2. Can one NSG be attached to multiple subnets?

**Yes.**

The same NSG can be associated with multiple subnets and/or NICs.

---

### Q3. Can one subnet have multiple NSGs?

**No.**

A subnet can have only **one** NSG associated with it.

---

### Q4. Can one NIC have multiple NSGs?

**No.**

A NIC can have only **one** NSG associated with it.

---

### Q5. If an NSG is attached to both a subnet and a NIC, which one is evaluated first?

* **Inbound:** Subnet NSG → NIC NSG → VM
* **Outbound:** VM → NIC NSG → Subnet NSG → Destination

---

# Summary

| Feature                          | NSG                                                                        |
| -------------------------------- | -------------------------------------------------------------------------- |
| Azure Firewall Type              | Stateless packet filtering based on rules (implemented using flow records) |
| OSI Layers                       | Layer 3 & Layer 4                                                          |
| Applied To                       | Subnet or NIC                                                              |
| Controls                         | Inbound & Outbound traffic                                                 |
| Rule Evaluation                  | Lowest priority number first                                               |
| Default Rules                    | Yes                                                                        |
| Multiple NSGs per Subnet         | No                                                                         |
| Multiple NSGs per NIC            | No                                                                         |
| Same NSG reused across resources | Yes                                                                        |

## NSG vs Azure Firewall

One important distinction is that an **NSG protects traffic to and from Azure resources within a VNet**, while **Azure Firewall** is a centralized, fully managed firewall service with advanced capabilities such as application rules, threat intelligence, and centralized logging. In most enterprise environments, you'll often see **NSGs used for micro-segmentation** and **Azure Firewall used for centralized perimeter security**. This combination provides layered network protection.
