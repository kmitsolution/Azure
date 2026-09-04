
# Azure Assignment 02 – Virtual Network, Subnet & CIDR

## 🎯 Objective

By completing this assignment, you should be able to:

* Understand Azure Virtual Network (VNet)
* Understand VNet address spaces
* Calculate CIDR ranges
* Create subnets
* Understand subnet-to-VM relationships
* Deploy VMs inside a VNet
* Understand private IP addresses
* Understand public vs private IP
* Understand how Azure resources connect to a VNet
* Understand Network Security Groups at a basic level
* Test connectivity between VMs
* Troubleshoot basic VNet connectivity

---

# Part 1 – Azure Virtual Network Basics

Answer the following questions.

### 1. What is an Azure Virtual Network?

Explain it in your own words.

### 2. Why do we need a VNet?

Give at least **3 real-world reasons**.

For example:

```text
Application
     ↓
Virtual Network
     ↓
Private communication
     ↓
Database
```

---

### 3. What is a VNet Address Space?

For example:

```text
10.0.0.0/16
```

Explain:

* What does `10.0.0.0` represent?
* What does `/16` represent?
* How many IP addresses are available?
* What is the first IP?
* What is the last IP?

---

# Part 2 – CIDR Assignment

This is an important exercise.

For the following CIDRs, calculate:

* Network address
* Number of IP addresses
* First usable IP
* Last usable IP
* Broadcast address concept

### Question 1

```text
10.0.0.0/24
```

### Question 2

```text
10.0.1.0/24
```

### Question 3

```text
10.0.0.0/16
```

### Question 4

```text
192.168.1.0/24
```

### Question 5

```text
172.16.0.0/20
```

### Question 6

```text
10.10.10.0/27
```

---

## ⭐ Challenge

You have:

```text
10.0.0.0/16
```

You need:

```text
Web Subnet       → 256 addresses
Application      → 256 addresses
Database         → 128 addresses
Management       → 64 addresses
```

Design appropriate subnet CIDRs.

For example:

```text
VNet
10.0.0.0/16
│
├── Web
│
├── Application
│
├── Database
│
└── Management
```

**Don't use overlapping address spaces.**

---

# Part 3 – VNet and Subnet Architecture

Design this architecture:

```text
                    Azure
                      │
                  VNet
               10.0.0.0/16
                      │
        ┌─────────────┼─────────────┐
        │             │             │
     WebSubnet    AppSubnet     DBSubnet
     /24           /24            /24
        │             │             │
      VM1            VM2           VM3
```

### Answer:

1. Why do we create multiple subnets?
2. Can a VNet have multiple subnets?
3. Can two subnets have overlapping CIDRs?
4. Can two VMs in different subnets communicate?
5. Can two VMs in the same subnet communicate?
6. Does each VM require its own subnet?
7. Can multiple VMs exist in the same subnet?

---

# Part 4 – Practical: Create Your First VNet

Go to:

**Azure Portal → Virtual Networks → Create**

Create:

```text
VNet Name:
vnet-training

Address Space:
10.0.0.0/16
```

Create these subnets:

```text
web-subnet
10.0.1.0/24

app-subnet
10.0.2.0/24

db-subnet
10.0.3.0/24
```

Your final architecture should be:

```text
vnet-training
10.0.0.0/16
│
├── web-subnet
│      10.0.1.0/24
│
├── app-subnet
│      10.0.2.0/24
│
└── db-subnet
       10.0.3.0/24
```

---

# Part 5 – Deploy VMs Inside the VNet

Now create two VMs.

### VM1

```text
Name: vm-web
Subnet: web-subnet
```

### VM2

```text
Name: vm-app
Subnet: app-subnet
```

Make sure both VMs belong to:

```text
vnet-training
```

Your architecture becomes:

```text
                vnet-training
                 10.0.0.0/16
                      │
          ┌───────────┴───────────┐
          │                       │
      web-subnet              app-subnet
      10.0.1.0/24              10.0.2.0/24
          │                       │
       vm-web                  vm-app
```

---

# Part 6 – Understand VM Networking

After creating the VMs, investigate the VM's networking.

For each VM find:

* Private IP
* Public IP
* NIC
* Subnet
* VNet
* NSG

Create a table:

| VM     | VNet | Subnet | Private IP | Public IP | NIC |
| ------ | ---- | ------ | ---------- | --------- | --- |
| vm-web | ?    | ?      | ?          | ?         | ?   |
| vm-app | ?    | ?      | ?          | ?         | ?   |

### Important Question

Does the VM itself directly connect to the VNet?

Think carefully.

The relationship is essentially:

```text
VM
 │
NIC
 │
Subnet
 │
VNet
```

This is an important Azure networking concept.

---

# Part 7 – Private IP vs Public IP

For `vm-web`, identify:

### Private IP

Example:

```text
10.0.1.4
```

### Public IP

Example:

```text
20.x.x.x
```

Answer:

1. Why does the VM need a private IP?
2. Why might it need a public IP?
3. Can VM1 communicate with VM2 using private IP?
4. Can you access the VM from the Internet using its private IP?
5. What happens if you remove the public IP?
6. Can a VM exist without a public IP?

---

# Part 8 – Basic VNet Connectivity Test

This is the most important practical section.

Suppose:

```text
vm-web
Private IP = 10.0.1.4

vm-app
Private IP = 10.0.2.4
```

From `vm-web`, test connectivity to `vm-app`.

### Windows

```powershell
ping 10.0.2.4
```

If ICMP isn't allowed, don't immediately conclude that networking is broken.

Test TCP instead.

For example:

```powershell
Test-NetConnection 10.0.2.4 -Port 3389
```

or:

```powershell
Test-NetConnection 10.0.2.4 -Port 80
```

### Linux

```bash
ping 10.0.2.4
```

and:

```bash
nc -vz 10.0.2.4 22
```

---

# Part 9 – Basic NSG Test

Now create an NSG and associate it with the appropriate subnet/NIC.

Initially, understand this architecture:

```text
VM1
 │
NIC
 │
NSG
 │
Subnet
 │
VNet
 │
NSG
 │
VM2
```

Create a rule that allows:

```text
Source: App subnet
Destination: Web subnet
Protocol: TCP
Port: 80
Action: Allow
```

Then test:

```text
VM2 ─────TCP 80─────> VM1
```

---

# Part 10 – Break It and Troubleshoot It 🔥

This is where the assignment becomes useful.

After confirming connectivity, intentionally create a problem.

For example:

```text
VM1 → VM2
TCP 80
```

should work.

Then create an NSG rule:

```text
Deny TCP 80
```

Test again.

### Questions

1. What changed?
2. Why did connectivity stop?
3. Where is the NSG attached?
4. Which rule is blocking traffic?
5. How would you troubleshoot this?

Then remove the deny rule and confirm connectivity works again.

---

# Part 11 – What Does "Create a Resource in a VNet" Mean?

This is a very important question.

A **VNet is not a container in which every Azure resource is physically placed**.

Different Azure services integrate with a VNet in different ways.

### VM

A VM uses:

```text
VM
 ↓
NIC
 ↓
Subnet
 ↓
VNet
```

### Azure Database / PaaS Service

A PaaS service may use mechanisms such as:

```text
Private Endpoint
      ↓
   Subnet
      ↓
     VNet
```

So when someone says:

> "Create the database inside the VNet"

you should ask **how that particular Azure service supports private networking**.

For example:

```text
                 VNet
                  │
       ┌──────────┴──────────┐
       │                     │
   VM Subnet          Private Endpoint Subnet
       │                     │
      VM              Private Endpoint
                             │
                        Azure Service
```

This concept will become extremely important when you learn **Azure PostgreSQL, Private Endpoint, Private DNS and Azure networking architecture**.

---

# Part 12 – Service Endpoint vs Private Endpoint

For now, only understand the basic difference.

### Service Endpoint

```text
VM
 │
Subnet
 │
Service Endpoint
 │
Azure Service
```

### Private Endpoint

```text
VM
 │
VNet
 │
Subnet
 │
Private Endpoint
 │
Private IP
 │
Azure Service
```

### Assignment

Research and explain:

1. What is a Service Endpoint?
2. What is a Private Endpoint?
3. What is the difference?
4. Which one provides a private IP inside the VNet?
5. Why would an organization prefer Private Endpoint for some workloads?

---

# Part 13 – Azure VNet Architecture Case Study

## 🏢 Company: ABC E-Commerce

The company has:

```text
Frontend
Application
Database
Management
```

Requirements:

* Internet users should access the frontend.
* Frontend should communicate with application servers.
* Application servers should communicate with the database.
* Database should not be directly accessible from the Internet.
* Administrators need management access.
* Different application tiers should use separate subnets.

### Your Task

Design the VNet.

Start with:

```text
10.10.0.0/16
```

Create:

```text
Frontend Subnet
Application Subnet
Database Subnet
Management Subnet
```

Then design:

```text
                    Internet
                       │
                       ↓
                Frontend Subnet
                       │
                       ↓
                Application Subnet
                       │
                       ↓
                 Database Subnet

                Management Subnet
                       │
                       ↓
                Admin Access
```

---

# Part 14 – Architecture Questions

Answer these based on your design.

### Q1

Why should the database not have a public IP?

### Q2

Why should frontend and database be in different subnets?

### Q3

Can frontend and application servers communicate even though they are in different subnets?

### Q4

What controls whether traffic is allowed?

### Q5

What happens if two subnets use:

```text
10.10.1.0/24
```

and

```text
10.10.1.0/24
```

?

### Q6

Can you create:

```text
10.10.1.0/25
```

inside:

```text
10.10.1.0/24
```

as a separate subnet?

Why/why not?

---

# Part 15 – Azure CLI Challenge

After completing the Portal exercise, recreate the basic networking architecture using Azure CLI.

Create:

```text
Resource Group:
rg-network-lab

VNet:
vnet-lab

Address Space:
10.20.0.0/16
```

Create:

```text
web-subnet       10.20.1.0/24
app-subnet       10.20.2.0/24
db-subnet        10.20.3.0/24
```

Then verify the configuration using Azure CLI.

Your objective is to be able to answer:

```text
What VNet exists?
What address space?
What subnets?
What CIDR does each subnet have?
```

---

# 🧪 Final VNet Test

Don't look at your notes while answering these.

### 1.

What is a VNet?

### 2.

What is a subnet?

### 3.

What is CIDR?

### 4.

What does `/24` mean?

### 5.

How many total IPv4 addresses are in `/24`?

### 6.

How many total IPv4 addresses are in `/16`?

### 7.

Can two subnets have overlapping CIDRs?

### 8.

Can VMs in different subnets communicate?

### 9.

What is the relationship between:

```text
VM → NIC → Subnet → VNet
```

### 10.

What is a private IP?

### 11.

What is a public IP?

### 12.

Can a VM work without a public IP?

### 13.

What is an NSG?

### 14.

Where can an NSG be associated?

### 15.

Why is ICMP ping not always a reliable test of Azure connectivity?

### 16.

What is a Private Endpoint?

### 17.

Does every Azure resource get "created inside" a VNet?

### 18.

How does a VM access a PaaS service privately?

### 19.

What is the difference between Service Endpoint and Private Endpoint?

### 20. 🔥 Architecture Question

You have:

```text
VNet: 10.0.0.0/16
```

Design subnets for:

```text
Web       → 500 hosts
Application → 500 hosts
Database  → 100 hosts
Management → 50 hosts
```

Choose the CIDR for each subnet and draw the final architecture.

---

## ⭐ Final Challenge

Build this completely in Azure:

```text
                         Azure
                           │
                    Resource Group
                           │
                       VNet /16
                    10.10.0.0/16
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
          ▼                ▼                 ▼
      Web Subnet       App Subnet       DB Subnet
      10.10.1.0/24    10.10.2.0/24     10.10.3.0/24
          │                │                 │
        VM-1              VM-2          Private Endpoint
                                             │
                                             ▼
                                        Azure PaaS
```

Then demonstrate:

**1.** VM-1 → VM-2 private connectivity
**2.** NSG blocking/allowing traffic
**3.** VM-2 → database private connectivity
**4.** No direct Internet access to the database
**5.** Identify private IPs, NICs, subnets and VNet
**6.** Explain the complete traffic flow

This assignment will give you the foundation for the next sequence of Azure networking lessons: **NSG → VNet Peering → UDR/Route Tables → NAT Gateway → Private Endpoint → Private DNS → Azure Load Balancer → Application Gateway → VNet architecture**.
