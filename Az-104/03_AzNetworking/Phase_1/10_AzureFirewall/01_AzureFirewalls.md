# Azure Firewall

Azure Firewall is a **fully managed, stateful, cloud-native network security service** provided by Microsoft.

Unlike an NSG, which is attached to a subnet or NIC, **Azure Firewall is deployed centrally** and protects multiple VNets or an entire Hub-Spoke architecture.

---

# Why Do We Need Azure Firewall?

Suppose you have:

```text
Internet
      │
      ▼
VNet
├── Web Subnet
├── App Subnet
└── DB Subnet
```

If you use only NSGs:

* Every subnet needs its own rules.
* Managing hundreds of NSGs becomes difficult.
* No centralized logging.
* No application filtering (FQDN filtering is limited).

Instead, enterprises deploy Azure Firewall.

```text
Internet
      │
Azure Firewall
      │
      ▼
Hub VNet
      │
Spoke VNets
```

All traffic passes through one firewall.

---

# Azure Firewall Features

* Stateful firewall
* Layer 3 and Layer 4 filtering
* Layer 7 application filtering
* DNAT
* SNAT
* FQDN filtering
* Threat Intelligence
* Logging
* High Availability
* Auto Scaling

---

# Azure Firewall Architecture

```text
Internet
      │
Public IP
      │
Azure Firewall
      │
Hub VNet
      │
──────────────
│      │      │
Dev   Test   Prod
```

---

# Difference Between NSG and Azure Firewall

| NSG                    | Azure Firewall                |
| ---------------------- | ----------------------------- |
| Applied to NIC/Subnet  | Centralized service           |
| L3/L4 filtering        | L3/L4 + L7 filtering          |
| Distributed            | Centralized                   |
| No DNAT/SNAT           | Supports DNAT/SNAT            |
| No Threat Intelligence | Threat Intelligence supported |
| No URL filtering       | FQDN/Application filtering    |

---

# Azure Firewall SKU

Azure offers three SKUs:

| SKU      | Features                            |
| -------- | ----------------------------------- |
| Basic    | Small environments                  |
| Standard | Most enterprise features            |
| Premium  | TLS Inspection, IDPS, URL filtering |

---

# Deployment

Azure Firewall requires a dedicated subnet.

The subnet name **must** be:

```text
AzureFirewallSubnet
```

Minimum recommended size:

```text
/26
```

---

# Traffic Flow

```text
Internet
      │
Azure Firewall
      │
VNet
      │
VM
```

Traffic is usually routed through the firewall using **User Defined Routes (UDRs)**.

---

# 1. DNAT (Destination Network Address Translation)

DNAT is used for **Inbound** traffic.

It translates a **Public IP** to a **Private IP**.

---

## Example

VM:

```text
Private IP

10.0.1.4
```

Firewall:

```text
Public IP

20.100.10.10
```

User connects:

```text
20.100.10.10:3389
```

Firewall performs:

```text
20.100.10.10

↓

10.0.1.4
```

This is DNAT.

---

## Architecture

```text
Internet

↓

20.100.10.10

↓

Azure Firewall

↓

10.0.1.4

↓

VM
```

---

## Example Rule

| Source   | Destination        | Port |
| -------- | ------------------ | ---- |
| Internet | Firewall Public IP | 3389 |

Translate to:

```text
10.0.1.4:3389
```

---

### Real Use Cases

* RDP
* SSH
* Web Servers
* Legacy Applications

---

# 2. SNAT (Source Network Address Translation)

SNAT is used for **Outbound** traffic.

Private IPs are translated into the firewall's Public IP.

---

Example

VM:

```text
10.0.1.4
```

Accesses:

```text
www.google.com
```

Firewall changes:

```text
10.0.1.4

↓

20.100.10.10
```

Google sees:

```text
20.100.10.10
```

instead of the private IP.

---

## Architecture

```text
VM

10.0.1.4

↓

Azure Firewall

↓

20.100.10.10

↓

Internet
```

---

## Why SNAT?

Private IPs cannot be routed on the Internet.

SNAT provides:

* Internet access
* One Public IP
* IP Whitelisting

---

# DNAT vs SNAT

| DNAT             | SNAT              |
| ---------------- | ----------------- |
| Inbound          | Outbound          |
| Public → Private | Private → Public  |
| RDP              | Internet Access   |
| SSH              | Windows Update    |
| Web Publishing   | Package Downloads |

---

# 3. Network Rules

Network Rules filter traffic based on:

* Source IP
* Destination IP
* Protocol
* Port

OSI Layer:

```text
Layer 3

Layer 4
```

---

## Example

Allow:

```text
Source

10.0.1.0/24

↓

Destination

10.0.2.0/24

↓

TCP

↓

443
```

---

### Supported Protocols

* TCP
* UDP
* ICMP

---

### Example

Allow SQL:

```text
Source

App Subnet

↓

Database

↓

1433
```

---

# 4. Application Rules

Application Rules work at:

```text
Layer 7
```

Instead of IPs, they use:

* FQDN
* URL
* HTTP
* HTTPS

---

Example:

Allow:

```text
github.com

microsoft.com

nuget.org
```

Deny:

```text
facebook.com

youtube.com
```

---

## Architecture

```text
VM

↓

Azure Firewall

↓

github.com

Allowed
```

---

## Example

Developers need:

```text
github.com

nuget.org

visualstudio.com
```

Everything else blocked.

---

# Network Rules vs Application Rules

| Network      | Application    |
| ------------ | -------------- |
| IP Based     | URL/FQDN Based |
| TCP/UDP/ICMP | HTTP/HTTPS     |
| L3/L4        | L7             |

---

# Rule Processing Order

Azure Firewall evaluates rules in this general order:

1. **DNAT rules**
2. **Network rules**
3. **Application rules**

Within a rule collection, rules are processed according to their configured priorities.

---

# 5. Threat Intelligence

Azure Firewall integrates with Microsoft's threat intelligence feeds.

It knows about:

* Malicious IPs
* Botnets
* Malware Hosts
* Command-and-Control Servers

---

Example

```text
VM

↓

Known Malware IP
```

Firewall blocks it.

---

## Modes

### Off

No checking.

---

### Alert

Traffic allowed.

Log generated.

---

### Alert and Deny

Traffic blocked.

Alert generated.

---

Example:

```text
Known Malware

↓

Azure Firewall

↓

Blocked
```

---

# 6. Firewall Manager

Firewall Manager is used to manage multiple Azure Firewalls centrally.

Suppose a company has:

```text
India

Firewall

US

Firewall

Europe

Firewall
```

Instead of configuring each firewall separately:

```text
Firewall Manager

↓

Policy

↓

All Firewalls
```

---

# Benefits

* Centralized management
* Shared policies
* Consistent security
* Hub-Spoke management
* Virtual WAN integration

---

# Enterprise Architecture

```text
Internet
      │
Azure Firewall
      │
Hub VNet
      │
───────────────
Dev
Test
Prod
```

Firewall Manager pushes one policy to all firewalls.

---

# Azure Firewall Policy

Rather than configuring each firewall individually, Microsoft recommends using **Firewall Policy**.

One policy contains:

* Network Rules
* Application Rules
* DNAT Rules
* Threat Intelligence settings

Example:

```text
Firewall Policy

↓

Firewall 1

Firewall 2

Firewall 3
```

---

# Enterprise Banking Example

```text
Internet

↓

Azure Firewall

↓

Application Gateway

↓

Web

↓

App

↓

Database
```

Rules:

Allow

* HTTPS
* SQL
* DNS

Deny

* Facebook
* YouTube
* Torrent Sites

---

# Interview Questions

## Q1. Does Azure Firewall replace NSG?

**No.**

They complement each other.

* **NSGs** provide distributed security at the subnet/NIC level.
* **Azure Firewall** provides centralized network security.

Most enterprises use **both**.

---

## Q2. Does Azure Firewall support Stateful inspection?

**Yes.**

It automatically tracks connection state, so return traffic for an allowed connection is permitted without requiring another rule.

---

## Q3. Difference between DNAT and SNAT?

| DNAT             | SNAT             |
| ---------------- | ---------------- |
| Inbound          | Outbound         |
| Public → Private | Private → Public |

---

## Q4. Can Azure Firewall filter websites?

**Yes.**

Using **Application Rules** based on FQDNs.

---

## Q5. Which service manages multiple Azure Firewalls?

**Azure Firewall Manager**.

---

# Hands-On Lab (Portal)

### Lab 1

* Create a Hub VNet.
* Create `AzureFirewallSubnet` (`/26`).
* Deploy Azure Firewall (Standard SKU).
* Assign a Standard Public IP.

### Lab 2

* Create a spoke VNet with a VM.
* Configure a User Defined Route (UDR) so all outbound traffic (`0.0.0.0/0`) goes to the Azure Firewall's private IP.

### Lab 3

* Create a **Network Rule** to allow DNS (53) and HTTPS (443).

### Lab 4

* Create an **Application Rule**:

  * Allow `github.com`
  * Allow `microsoft.com`
  * Deny other Internet sites (through policy design).

### Lab 5

* Create a **DNAT Rule**:

  * Firewall Public IP → VM RDP (3389) or SSH (22).

### Lab 6

* Enable **Threat Intelligence** in **Alert** mode.
* View logs in Azure Monitor or Log Analytics.

---

# Summary

| Feature                | Azure Firewall         |
| ---------------------- | ---------------------- |
| Firewall Type          | Managed, Stateful      |
| OSI Layers             | L3/L4 + L7             |
| DNAT                   | Yes                    |
| SNAT                   | Yes                    |
| Network Rules          | Yes                    |
| Application Rules      | Yes                    |
| Threat Intelligence    | Yes                    |
| Centralized Management | Yes (Firewall Manager) |
| Recommended Deployment | Hub VNet               |

