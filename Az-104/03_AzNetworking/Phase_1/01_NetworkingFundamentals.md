# Phase 1 – Networking Fundamentals (Foundation for Azure Networking)

Before learning Azure networking services like VNets, NSGs, VPN Gateway, or ExpressRoute, you must have strong networking fundamentals.

I would recommend spending around **2-3 weeks** on this phase.

---

# Module 1: Understand How Networks Work

## What is a Network?

A network is a group of devices connected together to communicate and share resources.

Example:

```text
Laptop  -------- Router -------- Internet
Phone    --------|
Server   --------|
```

Examples:

* Home Network
* Office Network
* Data Center Network
* Azure Virtual Network (VNet)

---

# Module 2: OSI Model (Very Important)

The OSI model has 7 layers.

```text
7 Application
6 Presentation
5 Session
4 Transport
3 Network
2 Data Link
1 Physical
```

---

## Layer 7 – Application Layer

Protocols:

* HTTP
* HTTPS
* FTP
* SMTP
* DNS

Examples:

* Chrome browser
* Outlook
* Postman

---

## Layer 6 – Presentation Layer

Responsible for:

* Encryption
* Compression
* Encoding

Examples:

* SSL/TLS
* JPEG
* MP3

---

## Layer 5 – Session Layer

Responsible for:

* Session establishment
* Session maintenance

Example:

Keeping a user logged into a website.

---

## Layer 4 – Transport Layer

Protocols:

* TCP
* UDP

Responsibilities:

* Reliability
* Sequencing
* Error handling

Examples:

* HTTPS → TCP
* Video Streaming → UDP

---

## Layer 3 – Network Layer

Responsible for:

* Routing
* IP Addressing

Devices:

* Routers

Protocols:

* IP
* ICMP

Examples:

* Ping
* Traceroute

---

## Layer 2 – Data Link Layer

Responsible for:

* MAC addressing

Devices:

* Switches

---

## Layer 1 – Physical Layer

Examples:

* Fiber cable
* Ethernet cable
* Wireless signals

---

# Interview Question

**At which layer does Azure NSG work?**

Answer:

```text
Layer 3 and Layer 4
```

---

# Module 3: TCP/IP Model

Azure networking mainly uses TCP/IP.

```text
Application
Transport
Internet
Network Access
```

Mapping:

| OSI                                  | TCP/IP         |
| ------------------------------------ | -------------- |
| Application + Presentation + Session | Application    |
| Transport                            | Transport      |
| Network                              | Internet       |
| Data Link + Physical                 | Network Access |

---

# Module 4: IPv4 Addressing

Every device requires an IP address.

Example:

```text
192.168.1.10
```

IPv4 contains:

```text
32 bits
4 octets
```

Example:

```text
192.168.1.10

11000000
10101000
00000001
00001010
```

---

# Public vs Private IP

## Private Ranges

```text
10.0.0.0/8

172.16.0.0/12

192.168.0.0/16
```

Private IPs cannot be routed on the Internet.

---

## Public IP

Globally unique.

Examples:

```text
20.40.120.10
52.168.11.20
```

Azure Public IP resources provide these addresses.

---

# Module 5: Binary Fundamentals

Networking heavily uses binary.

Example:

```text
255 = 11111111
192 = 11000000
224 = 11100000
240 = 11110000
248 = 11111000
252 = 11111100
254 = 11111110
```

You must remember these values.

---

# Module 6: Subnetting (Most Important)

This is the foundation of Azure VNets.

---

## CIDR Notation

Example:

```text
192.168.1.0/24
```

Meaning:

```text
24 bits → Network
8 bits → Hosts
```

---

## Host Calculation

Formula:

```text
Hosts = 2^(Host Bits) - 2
```

---

### Example 1

```text
192.168.1.0/24
```

Host bits:

```text
32 - 24 = 8
```

Hosts:

```text
2^8 - 2

256 - 2

254 hosts
```

---

### Example 2

```text
10.0.0.0/16
```

Hosts:

```text
2^16 -2

65534 hosts
```

---

# Common CIDRs

| CIDR | Hosts |
| ---- | ----- |
| /30  | 2     |
| /29  | 6     |
| /28  | 14    |
| /27  | 30    |
| /26  | 62    |
| /25  | 126   |
| /24  | 254   |
| /23  | 510   |
| /22  | 1022  |
| /21  | 2046  |
| /20  | 4094  |
| /16  | 65534 |

Memorize these.

---

# Module 7: Azure Reserved IP Addresses

Azure reserves 5 IPs in every subnet.

Example:

```text
10.0.1.0/24
```

Reserved:

```text
10.0.1.0
10.0.1.1
10.0.1.2
10.0.1.3
10.0.1.255
```

Available:

```text
10.0.1.4 – 10.0.1.254
```

This is frequently asked in interviews.

---

# Module 8: MAC Address

Example:

```text
00-15-5D-01-02-03
```

MAC Address:

* Layer 2 Address
* Physical Address
* 48 bits

Commands:

Windows:

```cmd
ipconfig /all
```

Linux:

```bash
ip addr
```

---

# Module 9: TCP vs UDP

---

## TCP

Characteristics:

* Connection oriented
* Reliable
* Acknowledgement based
* Slower

Examples:

* HTTP
* HTTPS
* SSH
* SQL

---

## UDP

Characteristics:

* Connectionless
* Faster
* No acknowledgements

Examples:

* DNS
* VoIP
* Video Streaming

---

# TCP Three-Way Handshake

```text
Client        Server

SYN -------->

<-------- SYN ACK

ACK -------->
```

---

# Module 10: Ports

Examples:

| Port  | Protocol   |
| ----- | ---------- |
| 20/21 | FTP        |
| 22    | SSH        |
| 23    | Telnet     |
| 25    | SMTP       |
| 53    | DNS        |
| 80    | HTTP       |
| 443   | HTTPS      |
| 1433  | SQL Server |
| 3306  | MySQL      |
| 3389  | RDP        |
| 5432  | PostgreSQL |

These are heavily used in Azure NSG rules.

---

# Module 11: DNS

DNS converts:

```text
www.google.com
```

into:

```text
142.250.x.x
```

Commands:

Windows:

```cmd
nslookup google.com
```

Linux:

```bash
dig google.com
```

---

# Module 12: Routing

Routing determines:

```text
How packets move from source to destination.
```

Command:

Windows:

```cmd
route print
```

Linux:

```bash
ip route
```

---

# Module 13: NAT

Private IPs cannot access Internet directly.

NAT converts:

```text
192.168.1.10
```

to

```text
20.40.10.11
```

Azure services using NAT:

* Azure NAT Gateway
* Azure Firewall
* Load Balancer outbound rules

---

# Module 14: Firewalls

A firewall filters traffic.

Examples:

```text
Allow 80
Allow 443
Deny Everything Else
```

Azure equivalents:

* NSG
* Azure Firewall
* WAF

---

# Module 15: VPN Concepts

Types:

### Site-to-Site VPN

```text
Office ⇄ Azure
```

### Point-to-Site VPN

```text
Laptop ⇄ Azure
```

### VNet-to-VNet VPN

```text
Azure VNet1 ⇄ Azure VNet2
```

---

# Practical Labs (Very Important)

## Lab 1

Install:

* VirtualBox
* Ubuntu VM
* Windows Server VM

Practice:

```bash
ip addr
ping
traceroute
ip route
```

---

## Lab 2

Create static IPs manually.

---

## Lab 3

Install Apache:

```bash
sudo apt install apache2
```

Access from another VM.

---

## Lab 4

Capture packets:

Install Wireshark.

Observe:

* DNS packets
* TCP handshake
* HTTP traffic

---

# Suggested Learning Sequence

```text
1. OSI Model
2. TCP/IP
3. Binary
4. IPv4
5. Subnetting
6. Routing
7. DNS
8. TCP/UDP
9. NAT
10. VPN
11. Firewalls
12. Packet Analysis
```

Once these concepts are clear, Azure networking becomes much easier because:

```text
VNet           → Network
Subnet         → Subnetting
NSG            → Firewall
VPN Gateway    → VPN
Route Table    → Routing
Private DNS    → DNS
NAT Gateway    → NAT
Application GW → Layer 7 Networking
```

For Azure engineers and AZ-700 preparation, **Subnetting, Routing, DNS, and TCP/IP are the most important fundamentals to master first.**
