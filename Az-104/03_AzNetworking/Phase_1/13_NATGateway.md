# Azure NAT Gateway — Detailed 

## 1. What is NAT?

**NAT = Network Address Translation.**

In Azure, a **NAT Gateway** provides **outbound Internet connectivity** for resources in a subnet.

For example, imagine a private VM:

```text
Private VM
10.0.2.4
```

The VM has only a **private IP**.

It wants to access:

```text
Internet
   |
   +-- ubuntu.com
   +-- microsoft.com
   +-- github.com
   +-- apt repositories
```

The private IP `10.0.2.4` cannot be used as the source address on the public Internet.

NAT Gateway solves this.

```text
Private VM
10.0.2.4
     |
     ↓
Private Subnet
     |
     ↓
NAT Gateway
     |
     ↓
Public IP
20.x.x.x
     |
     ↓
Internet
```

The Internet sees the **NAT Gateway's public IP**, not the VM's private IP.

---

# 2. Why Do We Need NAT Gateway?

Suppose you have 100 private VMs.

```text
PrivateSubnet
10.0.2.0/24

VM1 → 10.0.2.4
VM2 → 10.0.2.5
VM3 → 10.0.2.6
...
VM100
```

You don't want to assign a public IP to every VM.

Without NAT:

```text
VM1 → Public IP
VM2 → Public IP
VM3 → Public IP
...
VM100 → Public IP
```

This creates unnecessary public exposure and consumes public IP resources.

Instead:

```text
                     Internet
                         ↑
                         |
                   NAT Gateway
                   Public IP
                   20.x.x.x
                         ↑
                         |
                 Private Subnet
          +--------------+--------------+
          |              |              |
        VM1            VM2            VM3
     10.0.2.4       10.0.2.5       10.0.2.6
```

All the VMs can share the NAT Gateway's outbound public IP.

---

# 3. Very Important: NAT Gateway Is for Outbound Traffic

This is one of the most important AZ-104 points.

NAT Gateway provides:

> **Outbound Internet connectivity.**

For example:

```text
VM → Internet
```

is supported.

But NAT Gateway does **not** make the VM directly accessible from the Internet.

For example:

```text
Internet → VM
```

is **not** what NAT Gateway is designed to provide.

Think:

```text
NAT Gateway

OUTBOUND
VM ─────────────→ Internet
```

not:

```text
INBOUND
Internet ───────→ VM
```

---

# 4. Real-Life Example

Suppose we have a private Ubuntu VM.

```text
PrivateVM
Private IP:
10.0.2.4
```

It needs to run:

```bash
sudo apt update
```

Ubuntu needs to communicate with Internet repositories.

Without appropriate outbound connectivity, the VM may not be able to reach those public endpoints.

With NAT Gateway:

```text
PrivateVM
10.0.2.4
    |
    ↓
PrivateSubnet
10.0.2.0/24
    |
    ↓
NAT Gateway
    |
    ↓
Public IP
    |
    ↓
Internet
    |
    ↓
Ubuntu Repository
```

The repository sees the NAT Gateway's public IP.

---

# 5. Important Difference: Private IP Does NOT Mean No Internet

A VM can have:

```text
Private IP only
```

and still have:

```text
Outbound Internet access
```

provided an appropriate outbound connectivity mechanism exists.

For our lab:

```text
Private VM
     |
     ↓
NAT Gateway
     |
     ↓
Internet
```

This is a very common Azure architecture.

---

# 6. Our Lab Architecture

Let's create:

```text
Resource Group:
MYRG-India
```

VNet:

```text
MyVNet
10.0.0.0/16
```

Subnet:

```text
PrivateSubnet
10.0.2.0/24
```

VM:

```text
PrivateVM
```

NAT Gateway:

```text
MyNATGateway
```

Public IP:

```text
NATPublicIP
```

Architecture:

```text
                     INTERNET
                         ↑
                         |
                   Public IP
                   20.x.x.x
                         |
                         ↑
                   NAT Gateway
                  MyNATGateway
                         ↑
                         |
                  PrivateSubnet
                   10.0.2.0/24
                         |
                 +-------+-------+
                 |               |
                 ↓               ↓
             PrivateVM1      PrivateVM2
             10.0.2.4        10.0.2.5
```

---

# 7. What Happens When PrivateVM Goes to Internet?

Suppose:

```text
PrivateVM
10.0.2.4
```

runs:

```bash
curl https://www.microsoft.com
```

The flow is:

```text
PrivateVM
10.0.2.4
     |
     ↓
PrivateSubnet
10.0.2.0/24
     |
     ↓
NAT Gateway
     |
     ↓
NAT Public IP
     |
     ↓
Internet
     |
     ↓
Microsoft
```

The external website doesn't see:

```text
10.0.2.4
```

It sees the NAT Gateway public IP.

---

# 8. How Does Azure Know Which Subnet Uses NAT?

This is a very important point.

You don't attach NAT Gateway directly to a VM.

You associate the NAT Gateway with a **subnet**.

For example:

```text
NAT Gateway
     |
     ↓
PrivateSubnet
```

Therefore resources inside that subnet can use the NAT Gateway for outbound connectivity.

```text
PrivateSubnet
10.0.2.0/24
       |
       ↓
NAT Gateway
```

If you have:

```text
PublicSubnet
10.0.1.0/24
```

and don't associate the NAT Gateway with it, that subnet doesn't use this NAT Gateway.

---

# 9. Azure Portal — Step-by-Step

Let's assume you already have:

```text
Resource Group:
MYRG-India

VNet:
MyVNet

PrivateSubnet:
10.0.2.0/24
```

## Step 1 — Create Public IP

Go to:

```text
Azure Portal
   ↓
Public IP addresses
   ↓
Create
```

Configure:

```text
Resource Group:
MYRG-India

Name:
NATPublicIP

Region:
Central India

IP Version:
IPv4

SKU:
Standard

Assignment:
Static
```

Click:

**Review + create**

Then:

**Create**

---

# 10. Why Static Public IP?

We want the outbound public IP to remain predictable.

For example:

```text
NAT Gateway
     |
     ↓
20.1.2.3
```

If a third-party firewall allows our application's IP, we can give them:

```text
20.1.2.3
```

They can whitelist it.

This is one of the important real-world use cases of NAT Gateway.

---

# 11. Step 2 — Create NAT Gateway

Go to:

```text
Azure Portal
   ↓
NAT gateways
   ↓
Create
```

Select:

```text
Resource Group:
MYRG-India

NAT gateway name:
MyNATGateway

Region:
Central India
```

For public IP:

```text
Public IP:
NATPublicIP
```

Then:

```text
Review + create
```

and:

```text
Create
```

---

# 12. Step 3 — Associate NAT Gateway with Subnet

Open:

```text
MyNATGateway
```

Go to:

```text
Subnets
```

Click:

```text
Associate
```

Select:

```text
Virtual Network:
MyVNet

Subnet:
PrivateSubnet
```

Save.

Now the architecture becomes:

```text
MyVNet
   |
   ↓
PrivateSubnet
   |
   ↓
MyNATGateway
   |
   ↓
NATPublicIP
   |
   ↓
Internet
```

---

# 13. Very Important Concept

The association is:

```text
NAT Gateway
       ↓
    SUBNET
```

not:

```text
NAT Gateway
       ↓
      VM
```

So if you have:

```text
PrivateSubnet
     |
     +-- VM1
     +-- VM2
     +-- VM3
     +-- VM4
```

all of them can use the NAT Gateway.

```text
                     NAT Gateway
                         |
                         ↓
                   Public IP
                         |
                         ↓
                     Internet
                         ↑
                         |
                  PrivateSubnet
                  /     |     \
                VM1    VM2    VM3
```

---

# 14. Azure CLI — Create Public IP

Since you're using **Windows PowerShell**, use the following syntax.

Set variables:

```powershell
$RG = "MYRG-India"
$LOCATION = "centralindia"
$VNET = "MyVNet"
$SUBNET = "PrivateSubnet"
$NAT = "MyNATGateway"
$PUBLICIP = "NATPublicIP"
```

Create the public IP:

```powershell
az network public-ip create `
    --resource-group $RG `
    --name $PUBLICIP `
    --location $LOCATION `
    --sku Standard `
    --allocation-method Static
```

Check:

```powershell
az network public-ip show `
    --resource-group $RG `
    --name $PUBLICIP `
    --query ipAddress `
    --output tsv
```

For example:

```text
20.x.x.x
```

Your actual IP will be different.

---

# 15. Azure CLI — Create NAT Gateway

```powershell
az network nat gateway create `
    --resource-group $RG `
    --name $NAT `
    --location $LOCATION `
    --public-ip-addresses $PUBLICIP
```

Now:

```text
MyNATGateway
     |
     ↓
NATPublicIP
```

---

# 16. Azure CLI — Associate NAT Gateway with PrivateSubnet

This is the most important command:

```powershell
az network vnet subnet update `
    --resource-group $RG `
    --vnet-name $VNET `
    --name $SUBNET `
    --nat-gateway $NAT
```

Now:

```text
PrivateSubnet
      |
      ↓
MyNATGateway
      |
      ↓
NATPublicIP
      |
      ↓
Internet
```

---

# 17. Verify the Association

Run:

```powershell
az network vnet subnet show `
    --resource-group $RG `
    --vnet-name $VNET `
    --name $SUBNET `
    --query natGateway.id `
    --output tsv
```

You should get something similar to:

```text
/subscriptions/.../resourceGroups/MYRG-India/providers/Microsoft.Network/natGateways/MyNATGateway
```

---

# 18. Create Private VM

You can create an Ubuntu VM without a public IP.

For example:

```powershell
az vm create `
    --resource-group $RG `
    --name PrivateVM `
    --image Ubuntu2204 `
    --vnet-name $VNET `
    --subnet PrivateSubnet `
    --admin-username azureuser `
    --generate-ssh-keys `
    --public-ip-address ""
```

The important point is:

```text
--public-ip-address ""
```

We don't assign a public IP to the VM.

The VM gets something like:

```text
PrivateVM
Private IP:
10.0.2.4
```

---

# 19. Now How Does PrivateVM Reach Internet?

Architecture:

```text
                 INTERNET
                    ↑
                    |
               NAT Public IP
                    |
                    ↑
              NAT Gateway
                    ↑
                    |
             PrivateSubnet
              10.0.2.0/24
                    |
                    ↓
                PrivateVM
                 10.0.2.4
```

The VM has:

```text
Private IP = 10.0.2.4
```

but no:

```text
Public IP
```

Yet it can have outbound Internet access through NAT Gateway.

---

# 20. Testing From the VM

SSH access to a VM without a public IP requires another connectivity method, such as:

* Azure Bastion
* another jump VM
* VPN/ExpressRoute
* other private connectivity

Once you're inside the VM, run:

```bash
curl ifconfig.me
```

or:

```bash
curl https://api.ipify.org
```

You should see the **NAT Gateway public IP**, for example:

```text
20.x.x.x
```

It should match:

```text
NATPublicIP
```

This is an excellent demonstration.

---

# 21. Test Internet Connectivity

Inside Ubuntu:

```bash
curl https://www.microsoft.com
```

or:

```bash
curl https://google.com
```

You can also test:

```bash
sudo apt update
```

If the network configuration and outbound controls are correct, the VM can reach the Internet through NAT Gateway.

---

# 22. What About Inbound Traffic?

Suppose your NAT Gateway public IP is:

```text
20.1.2.3
```

Someone on the Internet tries:

```text
Internet
   |
   ↓
20.1.2.3
   |
   ↓
PrivateVM
```

That's **not the normal NAT Gateway use case**.

NAT Gateway is for:

```text
PrivateVM
    |
    ↓
NAT Gateway
    |
    ↓
Internet
```

not:

```text
Internet
    |
    ↓
NAT Gateway
    |
    ↓
PrivateVM
```

If you need inbound access to private applications, you would typically consider services such as:

* Azure Load Balancer
* Application Gateway
* Azure Firewall DNAT
* Front Door, depending on the application architecture

---

# 23. NAT Gateway vs Public IP on VM

This is another important AZ-104 comparison.

### Public IP on VM

```text
Internet
    ↕
Public IP
    ↕
VM
```

The VM has a public IP directly associated with its NIC.

### NAT Gateway

```text
Private VM
10.0.2.4
    |
    ↓
NAT Gateway
    |
    ↓
Public IP
    |
    ↓
Internet
```

The VM remains private.

---

# 24. NAT Gateway vs Load Balancer

Don't confuse these.

### NAT Gateway

Primarily:

```text
PRIVATE → INTERNET
```

Outbound connectivity.

### Load Balancer

Can provide:

```text
INTERNET
    ↓
Load Balancer
    ↓
VMs
```

for inbound application traffic, and Azure Load Balancer can also provide outbound connectivity depending on configuration.

---

# 25. NAT Gateway vs Azure Firewall

Both can provide outbound connectivity, but they have different purposes.

### NAT Gateway

```text
Private VM
    ↓
NAT Gateway
    ↓
Internet
```

Focused on scalable outbound SNAT.

### Azure Firewall

```text
Private VM
    ↓
Azure Firewall
    ↓
Internet
```

Firewall provides more extensive traffic filtering, logging and network security capabilities.

A common enterprise architecture might therefore be:

```text
Private VM
    |
    ↓
Azure Firewall
    |
    ↓
Internet
```

while NAT Gateway is used when the primary requirement is reliable/scalable outbound Internet SNAT.

---

# 26. Why NAT Gateway Is Better Than Giving Every VM a Public IP

Imagine 50 VMs:

```text
VM1 → Public IP
VM2 → Public IP
VM3 → Public IP
...
VM50 → Public IP
```

Instead:

```text
                  NAT Gateway
                      |
                 One/Few Public IPs
                      |
        +-------------+-------------+
        |             |             |
       VM1           VM2           VM3
        |             |             |
        +-------------+-------------+
                      |
               Private Subnet
```

Advantages include:

* VMs don't need individual public IPs.
* Centralized outbound connectivity.
* Predictable outbound public IP.
* Can scale to many resources in the subnet.
* Helps keep workloads private from unsolicited inbound Internet connections.

---

# 27. Important NAT Gateway Rule

NAT Gateway works at the **subnet level**.

Remember:

```text
NAT Gateway
     ↓
  SUBNET
     ↓
  VMs
```

not:

```text
NAT Gateway
     ↓
Individual VM
```

Therefore:

```text
PrivateSubnet
10.0.2.0/24
     |
     +-- VM1
     +-- VM2
     +-- VM3
     +-- VM4
```

can all use:

```text
MyNATGateway
```

if that NAT Gateway is associated with the subnet.

---

# 28. Multiple Subnets

Suppose you have:

```text
MyVNet
 |
 +-- PublicSubnet
 |
 +-- PrivateSubnet
 |
 +-- DatabaseSubnet
```

You can associate NAT Gateway with:

```text
PrivateSubnet
DatabaseSubnet
```

if those subnets require outbound Internet access.

```text
                  NAT Gateway
                       |
              +--------+--------+
              |                 |
       PrivateSubnet      DatabaseSubnet
              |                 |
            VM1              DB/VM
```

The NAT Gateway can serve multiple subnets in the VNet.

---

# 29. NAT Gateway and NSG

Suppose your subnet has an NSG.

```text
PrivateVM
    |
    ↓
NSG
    |
    ↓
NAT Gateway
    |
    ↓
Internet
```

The NSG can still control traffic.

For example:

```text
Outbound:
TCP 443 → Allow
```

If your NSG blocks the required outbound traffic, NAT Gateway doesn't magically override the NSG.

So remember:

```text
NSG = Traffic filtering
NAT = Address translation/outbound connectivity
```

They solve different problems.

---

# 30. NAT Gateway and Route Table

You may also hear:

> "Do I need a route table for NAT Gateway?"

For the basic NAT Gateway configuration:

**No custom route table is required.**

You associate:

```text
NAT Gateway
      ↓
Subnet
```

and Azure handles the appropriate outbound path.

If you have custom routing, Azure Firewall, NVA, forced tunneling, etc., then routing becomes a separate design consideration.

---

# 31. NAT Gateway + Private Endpoint

This is particularly useful because we've just discussed Private Endpoints.

Suppose your Private VM needs both:

1. Internet access for updates.
2. Private access to Storage.

You can have:

```text
                       MyVNet
                          |
             +------------+------------+
             |                         |
             ↓                         ↓
       PrivateSubnet          PrivateEndpointSubnet
             |                         |
         PrivateVM                Private Endpoint
             |                         |
             ↓                      10.0.3.5
        NAT Gateway                    |
             |                         ↓
             ↓                    Storage Account
         Internet
```

Now:

### Internet traffic

```text
PrivateVM
   ↓
NAT Gateway
   ↓
Internet
```

### Storage traffic

```text
PrivateVM
   ↓
Private Endpoint
   ↓
Storage
```

This is an excellent real-world Azure architecture.

---

# 32. The Complete Example

```text
                         INTERNET
                            ↑
                            |
                       NAT Public IP
                            |
                            ↑
                      NAT Gateway
                            ↑
                            |
                    PrivateSubnet
                    10.0.2.0/24
                            |
                       PrivateVM
                        10.0.2.4
                            |
               +------------+------------+
               |                         |
               |                         |
               ↓                         ↓
          Internet                 Private Endpoint
                                     10.0.3.5
                                         |
                                         ↓
                                  Storage Account
```

So the same VM can have:

```text
Internet access
       +
Private access to Azure Storage
```

without having a public IP itself.

---

# 33. AZ-104 Memory Trick

Remember:

### NAT Gateway

> **PRIVATE VM → PUBLIC INTERNET**

```text
PRIVATE → NAT → INTERNET
```

### Private Endpoint

> **PRIVATE VM → PRIVATE AZURE SERVICE**

```text
PRIVATE VM → PRIVATE ENDPOINT → AZURE SERVICE
```

### NSG

> **ALLOW / DENY TRAFFIC**

```text
TRAFFIC → NSG → ALLOW/DENY
```

### Azure Firewall

> **INSPECT AND CONTROL NETWORK TRAFFIC**

```text
TRAFFIC → FIREWALL → INTERNET
```

---

# 34. One-Line AZ-104 Definition

> **Azure NAT Gateway is a fully managed service that provides scalable and predictable outbound Internet connectivity for resources in an Azure virtual network subnet without requiring those resources to have individual public IP addresses.**

And the most important diagram to memorize:

```text
                 INTERNET
                     ↑
                     |
              NAT Public IP
                     ↑
                     |
                NAT Gateway
                     ↑
                     |
              PRIVATE SUBNET
                     ↑
                     |
                PRIVATE VM
```

**Memory trick:**

> **NAT = Private VM goes OUT.**

Whereas:

> **Private Endpoint = Private VM reaches an Azure service PRIVATELY.**
