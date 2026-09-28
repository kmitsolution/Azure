> **Service Endpoint = secure access from a subnet to an Azure service.**
> **Private Endpoint = gives the Azure service a private IP inside your VNet.**

## Service Endpoint vs Private Endpoint

| Feature                                 | Service Endpoint                             | Private Endpoint                                                                                                 |
| --------------------------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Main purpose                            | Secure access to Azure PaaS service          | Private connectivity to Azure PaaS service                                                                       |
| Configured on                           | **Subnet**                                   | **Subnet**                                                                                                       |
| Private IP created in VNet              | ❌ No                                         | ✅ Yes                                                                                                            |
| Azure service gets a private IP in VNet | ❌ No                                         | ✅ Through Private Endpoint                                                                                       |
| Uses a NIC in your VNet                 | ❌ No                                         | ✅ Yes                                                                                                            |
| Traffic path                            | VM → Azure backbone → Service                | VM → Private IP → Private Endpoint → Service                                                                     |
| DNS changes normally required           | ❌ No                                         | ✅ Private DNS commonly used                                                                                      |
| Can restrict public access to service   | Depends on service configuration             | Yes, commonly used with public network access disabled/restricted                                                |
| NSG involvement                         | NSG can control VM/subnet traffic            | NSG can also be used with the Private Endpoint NIC/subnet, subject to Private Endpoint network policies/settings |
| Cost                                    | No separate Private Endpoint resource charge | Private Endpoint has associated charges                                                                          |
| Example                                 | VM subnet → Storage Account                  | VM → `10.0.3.5` → Private Endpoint → Storage                                                                     |
| Memory trick                            | **SUBNET → SERVICE**                         | **PRIVATE IP → SERVICE**                                                                                         |

---

# 1. Service Endpoint

Suppose we have:

```text
MyVNet
10.0.0.0/16
    |
    ↓
PrivateSubnet
10.0.2.0/24
    |
    ↓
VM
10.0.2.4
    |
    ↓
Service Endpoint
Microsoft.Storage
    |
    ↓
Storage Account
```

You enable:

```text
Microsoft.Storage
```

on the subnet.

The important thing is:

**No private IP is created for the Storage Account inside your VNet.**

The Storage Account still has its Azure service endpoint.

The Service Endpoint essentially allows Azure to recognize that the traffic originates from your selected VNet/subnet and lets you apply the service's network access controls accordingly.

---

# 2. Private Endpoint

Now consider:

```text
MyVNet
10.0.0.0/16
    |
    +-- PublicSubnet
    |      |
    |   PublicVM
    |
    +-- PrivateSubnet
    |      |
    |   PrivateVM
    |
    +-- PrivateEndpointSubnet
           |
       Private Endpoint
           |
        10.0.3.5
           |
           ↓
     Storage Account
```

Here Azure creates a **Private Endpoint network interface** with a private IP such as:

```text
10.0.3.5
```

The VM can access the Storage Account through that private IP.

---

# 3. The Biggest Difference

This is the most important AZ-104 concept:

### Service Endpoint

```text
VM
 |
Subnet
 |
Service Endpoint
 |
Azure Storage
```

**No private IP for Storage inside your VNet.**

### Private Endpoint

```text
VM
 |
VNet
 |
10.0.3.5
 |
Private Endpoint
 |
Azure Storage
```

**Private IP exists inside your VNet.**

---

# 4. DNS Difference

This is another important difference.

### Service Endpoint

Your VM can continue using:

```text
mystorage.blob.core.windows.net
```

There is normally no need to create a Private DNS Zone just because you enabled a Service Endpoint.

---

### Private Endpoint

You normally use Private DNS.

For example:

```text
VM
 |
 | mystorage.blob.core.windows.net
 ↓
Private DNS
 |
 | resolves to
 ↓
10.0.3.5
 |
 ↓
Private Endpoint
 |
 ↓
Storage Account
```

So:

> **Private Endpoint + Private DNS = normal Azure hostname resolves to private IP.**

---

# 5. Example with Storage Account

Let's say:

```text
Storage Account:
mystorage123
```

## Service Endpoint

Configure the subnet:

```text
PrivateSubnet
       |
       ↓
Service Endpoints:
Microsoft.Storage
```

Then:

```text
VM
 |
PrivateSubnet
 |
Microsoft.Storage Service Endpoint
 |
Storage Account
```

There is no:

```text
10.0.3.5
```

representing Storage inside your VNet.

---

## Private Endpoint

Create:

```text
PrivateEndpointSubnet
10.0.3.0/24
```

Then:

```text
Private Endpoint
       |
       ↓
10.0.3.5
       |
       ↓
Storage Account
```

Now the architecture is:

```text
MyVNet
 |
 +-- PublicSubnet
 |
 +-- PrivateSubnet
 |       |
 |    PrivateVM
 |
 +-- PrivateEndpointSubnet
         |
      10.0.3.5
         |
    Private Endpoint
         |
         ↓
   Storage Account
```

---

# 6. Security Perspective

Think of the two technologies this way.

### Service Endpoint

You are basically saying:

> "Only traffic coming from these selected VNets/subnets should be allowed to access my Storage Account."

For example, Storage networking can be configured to allow a particular VNet/subnet.

```text
VM
 ↓
VNet/Subnet
 ↓
Service Endpoint
 ↓
Storage
```

---

### Private Endpoint

You are saying:

> "I want a private network interface/IP in my VNet through which my applications can access the Storage Account."

```text
VM
 ↓
Private IP
 ↓
Private Endpoint
 ↓
Storage
```

---

# 7. Can I Disable Public Access?

This is an important practical difference.

With a **Private Endpoint**, a common architecture is:

```text
Internet
   |
   X
Storage Account
   ↑
   |
Private Endpoint
   ↑
   |
MyVNet
```

You can configure the Storage Account's networking so that public network access is restricted or disabled, while your applications use the Private Endpoint.

With a Service Endpoint, the Azure service still uses its service endpoint; it isn't given a private IP in your VNet.

---

# 8. NSG Difference

Don't confuse **Service Endpoint** with **NSG**.

Service Endpoint:

```text
Subnet
   |
   ↓
Service Endpoint
   |
   ↓
Azure Service
```

NSG:

```text
Subnet/NIC
   |
   ↓
NSG
   |
   +-- Allow
   +-- Deny
```

Private Endpoint:

```text
Subnet
   |
   ↓
Private Endpoint NIC
   |
Private IP
   |
   ↓
Azure Service
```

You can use NSGs as part of the overall network security design in both architectures, but the **purpose of NSG is traffic filtering**, whereas the purpose of the endpoint is connectivity to the Azure service.

---

# 9. Very Easy Memory Trick

Remember these three questions:

### Service Endpoint

**"Which subnet is accessing the service?"**

```text
SUBNET → SERVICE
```

### Private Endpoint

**"Which private IP should I use to access the service?"**

```text
PRIVATE IP → SERVICE
```

### NSG

**"Should I allow or deny this traffic?"**

```text
ALLOW / DENY → TRAFFIC
```

---

# 10. One Final Diagram

### Service Endpoint

```text
              MyVNet
                 |
             Subnet
                 |
                VM
                 |
                 ↓
        Service Endpoint
                 |
                 ↓
          Azure Storage
                 
       No private IP
       for Storage in VNet
```

### Private Endpoint

```text
              MyVNet
                 |
       PrivateEndpointSubnet
                 |
                 ↓
         Private Endpoint
                 |
              10.0.3.5
                 |
                 ↓
          Azure Storage

       Private IP exists
       inside your VNet
```

## One-line exam answer

> **Service Endpoint provides subnet-level access from a VNet to supported Azure services without giving the service a private IP in your VNet, whereas Private Endpoint creates a network interface with a private IP in your VNet to privately access the Azure service.**

That **"no private IP vs private IP"** distinction is the single most important thing to remember for AZ-104.
