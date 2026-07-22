# Phase 1: Networking Fundamentals (Must Know First)

Before touching Azure services, make sure these concepts are very clear.

## Topics to Learn

1. OSI Model (7 Layers)
2. TCP/IP Model
3. IPv4 Addressing
4. Subnetting (Very Important)
5. CIDR Notation
6. Public vs Private IP
7. NAT
8. DNS
9. Routing Tables
10. TCP vs UDP
11. Ports and Protocols
12. Firewalls
13. VPN Concepts
14. Load Balancing Basics

### Practice

Install:

* VirtualBox
* Ubuntu VMs
* Windows Server VM

Practice:

* Configure static IPs
* Create subnets
* Configure DNS
* Test routing using ping, traceroute
* Use Wireshark

---

# Phase 2: Azure Networking Basics

## 1. Azure Regions and Availability Zones

Understand:

* Regions
* Region Pairs
* Availability Zones
* Edge Locations

---

## 2. Virtual Networks (VNet)

Topics:

* What is VNet
* Address Spaces
* Multiple Address Spaces
* CIDR Planning
* Reserved Azure IPs
* VNet Limits

### Hands-on

```bash
az network vnet create
```

Create:

* Dev VNet
* Prod VNet

---

## 3. Subnets

Topics:

* Subnet creation
* IP planning
* Service Endpoints
* Delegated Subnets

Practice:

```bash
az network vnet subnet create
```

Create:

* Web subnet
* App subnet
* DB subnet

---

## 4. Network Interfaces (NIC)

Learn:

* Primary IP
* Secondary IP
* Multiple NICs
* Accelerated Networking

---

## 5. IP Addressing

Topics:

### Public IP

* Static
* Dynamic
* SKU Standard vs Basic

### Private IP

* Dynamic
* Static

Practice attaching multiple IPs to VMs.

---

# Phase 3: Network Security

## Network Security Groups (NSG)

Topics:

* Inbound Rules
* Outbound Rules
* Priorities
* ASG Integration
* Default Rules

Hands-on:

* Allow HTTP
* Allow SSH
* Deny Everything Else

---

## Application Security Groups (ASG)

Learn:

* Group-based security
* Micro segmentation

---

## Azure Firewall

Topics:

* DNAT
* SNAT
* Network Rules
* Application Rules
* Threat Intelligence
* Firewall Manager

Architecture practice:

```
Internet
    ↓
Azure Firewall
    ↓
Application
```

---

## Azure DDoS Protection

Learn:

* Basic
* Standard
* Attack types

---

# Phase 4: Routing

## User Defined Routes (UDR)

Topics:

* System Routes
* Custom Routes
* Next Hop Types

Hands-on:

Route traffic through firewall VM.

---

## Route Server

Understand BGP integration.

---

# Phase 5: Connectivity Services

## VNet Peering

Topics:

* Local Peering
* Global Peering
* Transitive Limitation
* Gateway Transit

Lab:

```
VNet1 ←→ VNet2 ←→ VNet3
```

Check connectivity.

---

## VPN Gateway

Topics:

### Site-to-Site VPN

### Point-to-Site VPN

### VNet-to-VNet VPN

Learn:

* IPSec
* IKE
* Active-Active Gateway

---

## ExpressRoute

Topics:

* What is ExpressRoute
* Circuit
* Peering Types
* FastPath
* Global Reach

Architecture:

```
On-Prem
     ↓
ExpressRoute
     ↓
Azure
```

---

# Phase 6: Name Resolution

## Azure DNS

Topics:

* Public Zones
* Private DNS Zones
* Auto Registration

Practice:

```bash
nslookup
dig
Resolve-DnsName
```

---

## DNS Private Resolver

Learn:

* Inbound Endpoint
* Outbound Endpoint
* Hybrid DNS

---

# Phase 7: Load Balancing Services

This is one of the biggest networking topics.

---

## Azure Load Balancer (Layer 4)

Topics:

* Internal LB
* Public LB
* Health Probes
* HA Ports

Lab:

Deploy two VMs behind LB.

---

## Application Gateway (Layer 7)

Topics:

* URL Routing
* SSL Termination
* Rewrite Rules
* Session Affinity
* WAF

Architecture:

```
Internet
      ↓
Application Gateway
      ↓
Web Servers
```

---

## Traffic Manager

Topics:

* DNS based balancing
* Failover
* Performance routing
* Geographic routing

---

## Front Door

Topics:

* Global Load Balancer
* CDN Integration
* WAF
* Anycast

Architecture:

```
Users Worldwide
        ↓
Azure Front Door
        ↓
Regional Applications
```

---

# Phase 8: Private Connectivity

## Service Endpoints

Understand:

* Microsoft Backbone
* Limitations

---

## Private Endpoints (Very Important)

Services:

* Storage
* SQL
* Key Vault
* Container Registry

Lab:

Create Storage Account with:

```text
Public Access = Disabled
Private Endpoint = Enabled
```

Access only through VNet.

---

# Phase 9: Hybrid Networking

Topics:

* Hub and Spoke
* Transit VNet
* Shared Services VNet
* DMZ Architecture

Design:

```text
Hub
 ├── Firewall
 ├── VPN Gateway
 └── DNS

Spokes
 ├── Dev
 ├── Test
 └── Production
```

This architecture is heavily asked in interviews.

---

# Phase 10: Monitoring and Troubleshooting

## Network Watcher

Learn:

* Topology
* Connection Troubleshoot
* NSG Flow Logs
* Packet Capture
* Next Hop
* IP Flow Verify

---

## Azure Monitor

Topics:

* Metrics
* Alerts
* Log Analytics

---

# Phase 11: Advanced Topics

## Virtual WAN

Topics:

* Secure Hub
* Branch Connectivity
* SD-WAN Integration

---

## Azure Firewall Manager

---

## NAT Gateway

Topics:

* Outbound Connectivity
* SNAT Port Exhaustion

---

## Azure Bastion

Topics:

* Secure RDP/SSH
* Private Connectivity

---

## IPv6 in Azure

---

# Phase 12: Networking for PaaS Services

Learn networking integration for:

* App Service
* Azure Kubernetes Service (AKS)
* Azure Container Apps
* Storage Account
* SQL Database
* Key Vault

Since you are also learning Azure Container Apps and AKS, networking becomes extremely important.

---

# Suggested Learning Order (12 Weeks)

| Week | Topics                               |
| ---- | ------------------------------------ |
| 1    | Networking Fundamentals + Subnetting |
| 2    | VNets + Subnets + NIC + IP           |
| 3    | NSG + ASG + Azure Firewall           |
| 4    | Routing + UDR                        |
| 5    | Peering                              |
| 6    | VPN Gateway                          |
| 7    | ExpressRoute                         |
| 8    | DNS                                  |
| 9    | Load Balancers + App Gateway         |
| 10   | Front Door + Traffic Manager         |
| 11   | Private Endpoint + Service Endpoint  |
| 12   | Hub-Spoke + Virtual WAN + Monitoring |

---

# Best Hands-on Projects

### Project 1

Three-tier application:

```text
Internet
    ↓
Application Gateway
    ↓
Web Subnet
    ↓
App Subnet
    ↓
DB Subnet
```

---

### Project 2

Hub-Spoke Architecture

```text
OnPrem
   ↓
VPN Gateway
   ↓
Hub VNet
   ↓
Spokes
```

---

### Project 3

Private PaaS Architecture

```text
AKS
 ↓
Private Endpoint
 ↓
Storage
 ↓
Key Vault
```

---

# Certifications where these topics are useful

* AZ-104
* AZ-700 (Azure Network Engineer Associate)
* AZ-305
* AKS Networking
* Azure Solutions Architect interviews

For deep networking knowledge, I strongly recommend eventually focusing on **AZ-700**, because it covers almost every Azure networking service in detail and is considered the best certification specifically for Azure Networking.
