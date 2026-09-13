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
