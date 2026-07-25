# Azure Application Security Groups (ASG)

Application Security Groups (ASGs) make NSG rules easier to manage by allowing you to **group virtual machines based on their role** instead of writing rules using IP addresses.

Think of ASGs as **logical groups** of VMs.

---

# Why Do We Need ASGs?

Imagine you have 100 VMs.

Without ASGs, your NSG rule might look like this:

```text
Allow

10.0.1.4
10.0.1.5
10.0.1.6
10.0.1.7
10.0.1.8
...

↓

10.0.2.4
10.0.2.5
10.0.2.6

Port 443
```

Now suppose a new VM is created.

```text
10.0.1.20
```

You must manually update all NSG rules.

This quickly becomes difficult to manage.

---

# Solution: Application Security Groups

Instead of IP addresses, you create groups.

Example:

```text
Web-ASG

App-ASG

DB-ASG
```

Now your NSG rule becomes:

```text
Source      : Web-ASG

Destination : App-ASG

Port        : 443

Action      : Allow
```

No IP addresses are used.

---

# Real Example

Suppose you have:

```text
Web Servers

VM1
VM2
VM3
```

All belong to:

```text
Web-ASG
```

Similarly:

```text
App1
App2
```

belong to:

```text
App-ASG
```

Database:

```text
SQL1
```

belongs to:

```text
DB-ASG
```

---

# Architecture

```text
Internet
      │
Application Gateway
      │
───────────────
Web-ASG
│
├── VM1
├── VM2
└── VM3
      │
───────────────
App-ASG
│
├── App1
└── App2
      │
───────────────
DB-ASG
│
└── SQL1
```

---

# NSG Rules Using ASGs

### Rule 1

```text
Source

Internet

↓

Destination

Web-ASG

Port 443

Allow
```

---

### Rule 2

```text
Web-ASG

↓

App-ASG

Port 443

Allow
```

---

### Rule 3

```text
App-ASG

↓

DB-ASG

Port 1433

Allow
```

---

### Rule 4

```text
Any

↓

DB-ASG

Port *

Deny
```

---

# Advantages

Without ASGs:

```text
Allow

10.0.1.4

10.0.1.5

10.0.1.6
```

With ASGs:

```text
Allow

Web-ASG
```

Much simpler.

---

# Dynamic Membership

Suppose you add:

```text
VM4
```

Simply assign it to:

```text
Web-ASG
```

The existing NSG rules automatically apply.

No rule modification is needed.

---

# How ASG Works

Step 1

Create ASG:

```text
Web-ASG
```

---

Step 2

Associate VM NICs.

```text
VM1

↓

NIC

↓

Web-ASG
```

---

Step 3

Reference ASG in NSG rules.

```text
Source

Web-ASG

↓

Destination

App-ASG
```

Azure automatically resolves the VM IP addresses.

---

# CLI Example

## Create ASG

```bash
az network asg create \
   -g demo-rg \
   -n Web-ASG
```

---

## Create App ASG

```bash
az network asg create \
   -g demo-rg \
   -n App-ASG
```

---

## Associate NIC with ASG

```bash
az network nic ip-config update \
   -g demo-rg \
   --nic-name webnic \
   -n ipconfig1 \
   --application-security-groups Web-ASG
```

---

# Create NSG Rule

```bash
az network nsg rule create \
   -g demo-rg \
   --nsg-name web-nsg \
   -n Allow-Web-To-App \
   --priority 100 \
   --source-asgs Web-ASG \
   --destination-asgs App-ASG \
   --destination-port-ranges 443 \
   --protocol Tcp \
   --access Allow
```

---

# ASG Scope

An ASG belongs to a **single VNet**.

Example:

```text
Central India

VNet-A

Web-ASG
```

You cannot directly use the same ASG for VMs in another VNet.

---

# Micro Segmentation

This is one of the biggest reasons ASGs are used.

## What is Micro Segmentation?

Micro segmentation means dividing your network into **small, secure segments** so that each application tier communicates **only with the tiers it needs**.

Instead of allowing:

```text
Everyone

↓

Everyone
```

you allow only:

```text
Web

↓

App

↓

Database
```

---

## Without Micro Segmentation

```text
Web VM

↓

Database
```

Allowed.

App VM

↓

Database

```

Allowed.

Any VM

↓

Any VM
```

Allowed.

This is not secure.

---

## With Micro Segmentation

```text
Internet
      │
Web-ASG
      │
Port 443
      │
App-ASG
      │
Port 1433
      │
DB-ASG
```

Only approved communication paths exist.

---

# Enterprise Banking Example

Imagine an online banking application.

```text
Internet
      │
Application Gateway
      │
──────────────
Web-ASG
      │
HTTPS
      │
──────────────
App-ASG
      │
SQL 1433
      │
──────────────
DB-ASG
```

### Allowed Traffic

| Source   | Destination | Port |
| -------- | ----------- | ---- |
| Internet | Web-ASG     | 443  |
| Web-ASG  | App-ASG     | 443  |
| App-ASG  | DB-ASG      | 1433 |

---

### Denied Traffic

```text
Internet

↓

DB-ASG
```

Denied.

---

```text
Internet

↓

App-ASG
```

Denied.

---

```text
Web-ASG

↓

DB-ASG
```

Denied.

---

# Why Is This More Secure?

If a web server is compromised, the attacker **cannot directly connect to the database** because the NSG rules only allow:

```text
Web-ASG

↓

App-ASG
```

and

```text
App-ASG

↓

DB-ASG
```

This limits lateral movement inside the network.

---

# Real Azure Example

```text
VNet

Web Subnet

VM1
VM2

↓

Web-ASG

-------------------

App Subnet

VM3
VM4

↓

App-ASG

-------------------

DB Subnet

SQL VM

↓

DB-ASG
```

Notice that **ASGs are based on application roles, not subnet boundaries**. Two VMs in the same subnet can belong to different ASGs if they serve different purposes.

---

# ASG vs NSG

Students often confuse these.

| NSG                        | ASG                          |
| -------------------------- | ---------------------------- |
| Contains security rules    | Contains VM group membership |
| Filters traffic            | Organizes VMs logically      |
| Applied to Subnets or NICs | Referenced inside NSG rules  |
| Acts like a firewall       | Acts like a label or group   |

An easy way to remember it is:

* **NSG decides *what traffic is allowed or denied*.**
* **ASG decides *which VMs belong to a particular application group*.**

---

# Interview Questions

### Q1. Does ASG replace NSG?

**No.**

ASGs work **with** NSGs. You reference ASGs inside NSG rules.

---

### Q2. Can one VM belong to multiple ASGs?

**Yes.** A NIC's IP configuration can be associated with multiple ASGs, allowing the VM to participate in multiple logical application groups.

---

### Q3. Can ASGs contain VMs from different VNets?

**No.**

ASGs are scoped to a single VNet.

---

### Q4. Can ASGs span multiple Azure regions?

**No.**

Since ASGs are VNet-scoped, and VNets are regional, ASGs are also effectively regional.

---

# Summary

| Feature                     | ASG                      |
| --------------------------- | ------------------------ |
| Purpose                     | Logical grouping of VMs  |
| Based On                    | Application role         |
| Uses IP addresses?          | No                       |
| Used Inside                 | NSG rules                |
| Supports Micro Segmentation | Yes                      |
| Simplifies Rule Management  | Yes                      |
| Regional                    | Yes (through VNet scope) |

## Best Practice

In enterprise environments, name ASGs by application tier rather than by subnet, for example:

* `asg-web`
* `asg-api`
* `asg-db`
* `asg-management`

This keeps your security policy aligned with the application architecture rather than the network layout, making it much easier to scale and maintain over time.
