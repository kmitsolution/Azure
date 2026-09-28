# 1. What is a Private Endpoint?

An **Azure Private Endpoint** is a network interface that gives an Azure PaaS service, such as an Azure Storage Account, a **private IP address from your VNet**.

The VM can then access the Azure service through this private IP.

The most important point is:

> **Private Endpoint creates a private IP address inside your VNet for accessing an Azure PaaS service.**

For example:

```text
                    Azure VNet
                 10.0.0.0/16
                       |
        +--------------+--------------+
        |              |              |
        ↓              ↓              ↓
 PublicSubnet    PrivateSubnet   PrivateEndpointSubnet
 10.0.1.0/24     10.0.2.0/24       10.0.3.0/24
        |              |              |
       VM             VM         Private Endpoint
                                      |
                                  10.0.3.5
                                      |
                                      ↓
                               Storage Account
```

---

# 2. Our Example

We will create the following architecture.

### Resource Group

```text
MYRG-India
```

### VNet

```text
MyVNet
10.0.0.0/16
```

### Subnets

```text
PublicSubnet
10.0.1.0/24

PrivateSubnet
10.0.2.0/24

PrivateEndpointSubnet
10.0.3.0/24
```

### Resources

```text
PublicVM
    ↓
PublicSubnet

PrivateVM
    ↓
PrivateSubnet

StoragePrivateEndpoint
    ↓
PrivateEndpointSubnet
```

The final architecture:

```text
                         MyVNet
                      10.0.0.0/16
                           |
          +----------------+----------------+
          |                |                |
          ↓                ↓                ↓
   PublicSubnet      PrivateSubnet   PrivateEndpointSubnet
    10.0.1.0/24       10.0.2.0/24       10.0.3.0/24
          |                |                |
          ↓                ↓                ↓
      PublicVM         PrivateVM      Private Endpoint
                                          |
                                      10.0.3.5
                                          |
                                          ↓
                                   Storage Account
```

---

# 3. Why Three Subnets?

We are using three subnets to make the architecture easy to understand.

## PublicSubnet

Contains our demonstration VM:

```text
PublicSubnet
     |
  PublicVM
```

## PrivateSubnet

Contains an internal/private VM:

```text
PrivateSubnet
     |
  PrivateVM
```

## PrivateEndpointSubnet

Contains the Private Endpoint:

```text
PrivateEndpointSubnet
          |
    Private Endpoint
          |
       10.0.3.5
```

The Private Endpoint receives its private IP from this subnet.

---

# 4. What Problem Does Private Endpoint Solve?

Suppose our VM needs to access an Azure Storage Account.

Without a Private Endpoint:

```text
VM
 |
 |
Public Storage Endpoint
 |
 |
Storage Account
```

The Storage Account has a normal Azure service endpoint such as:

```text
mystorageaccount.blob.core.windows.net
```

Now suppose the requirement is:

> "I want my VM to access the Storage Account through a private IP inside my VNet."

We create a Private Endpoint.

Now the architecture becomes:

```text
PublicVM
10.0.1.4
    |
    |
    ↓
MyVNet
    |
    |
Private Endpoint
10.0.3.5
    |
    |
    ↓
Storage Account
```

The Storage Account now has a **private entry point inside the VNet**.

---

# 5. What Exactly Is the Private Endpoint?

A Private Endpoint is essentially a **network interface with a private IP address** associated with an Azure PaaS service.

Conceptually:

```text
Private Endpoint
       |
       +-- Network Interface
       |
       +-- Private IP
             10.0.3.5
       |
       +-- Connected to Storage Account
```

So when we create:

```text
PrivateEndpointSubnet
10.0.3.0/24
```

Azure can assign:

```text
10.0.3.5
```

to the Private Endpoint.

---

# 6. How Does the VM Connect to Storage?

Suppose:

```text
PublicVM
10.0.1.4
```

needs to access:

```text
Storage Account
```

The application normally uses the Storage DNS name:

```text
mystorageaccount.blob.core.windows.net
```

With Private DNS configured, the name resolves to the private IP of the Private Endpoint.

For example:

```text
mystorageaccount.blob.core.windows.net
                |
                ↓
           Private DNS
                |
                ↓
            10.0.3.5
                |
                ↓
        Private Endpoint
                |
                ↓
        Storage Account
```

The VM doesn't normally need to know the private IP.

DNS handles the resolution.

---

# 7. Complete Architecture

```text
                         Azure
                           |
                    Resource Group
                      MYRG-India
                           |
                         MyVNet
                      10.0.0.0/16
                           |
       +-------------------+-------------------+
       |                   |                   |
       ↓                   ↓                   ↓
 PublicSubnet        PrivateSubnet      PrivateEndpointSubnet
 10.0.1.0/24         10.0.2.0/24           10.0.3.0/24
       |                   |                   |
       ↓                   ↓                   ↓
   PublicVM            PrivateVM        Private Endpoint
   10.0.1.4                                  |
                                             |
                                          10.0.3.5
                                             |
                                             ↓
                                      Storage Account
```

---

# 8. Azure Portal — Create the VNet

Go to:

```text
Azure Portal
    ↓
Virtual Networks
    ↓
Create
```

Select:

```text
Resource Group:
MYRG-India

Virtual Network:
MyVNet

Region:
Central India

Address Space:
10.0.0.0/16
```

---

# 9. Create PublicSubnet

Create:

```text
Subnet Name:
PublicSubnet

Address Range:
10.0.1.0/24
```

Architecture:

```text
MyVNet
  |
  +-- PublicSubnet
        10.0.1.0/24
```

---

# 10. Create PrivateSubnet

Create:

```text
Subnet Name:
PrivateSubnet

Address Range:
10.0.2.0/24
```

Architecture:

```text
MyVNet
  |
  +-- PrivateSubnet
        10.0.2.0/24
```

---

# 11. Create PrivateEndpointSubnet

Create:

```text
Subnet Name:
PrivateEndpointSubnet

Address Range:
10.0.3.0/24
```

Architecture:

```text
MyVNet
  |
  +-- PrivateEndpointSubnet
        10.0.3.0/24
```

Final VNet:

```text
MyVNet
10.0.0.0/16
   |
   +-- PublicSubnet
   |      10.0.1.0/24
   |
   +-- PrivateSubnet
   |      10.0.2.0/24
   |
   +-- PrivateEndpointSubnet
          10.0.3.0/24
```

---

# 12. Azure CLI — Create VNet

Since we are using Windows, the following commands use **Azure CLI from PowerShell**.

Set variables:

```powershell
$RG = "MYRG-India"
$LOCATION = "centralindia"
$VNET = "MyVNet"
```

Create VNet:

```powershell
az network vnet create `
    --resource-group $RG `
    --name $VNET `
    --location $LOCATION `
    --address-prefixes 10.0.0.0/16
```

---

# 13. Create PublicSubnet

```powershell
az network vnet subnet create `
    --resource-group $RG `
    --vnet-name $VNET `
    --name PublicSubnet `
    --address-prefixes 10.0.1.0/24
```

---

# 14. Create PrivateSubnet

```powershell
az network vnet subnet create `
    --resource-group $RG `
    --vnet-name $VNET `
    --name PrivateSubnet `
    --address-prefixes 10.0.2.0/24
```

---

# 15. Create PrivateEndpointSubnet

```powershell
az network vnet subnet create `
    --resource-group $RG `
    --vnet-name $VNET `
    --name PrivateEndpointSubnet `
    --address-prefixes 10.0.3.0/24
```

Verify:

```powershell
az network vnet subnet list `
    --resource-group $RG `
    --vnet-name $VNET `
    --output table
```

You should see:

```text
Name
-------------------------
PublicSubnet
PrivateSubnet
PrivateEndpointSubnet
```

---

# 16. Create Storage Account

Set the Storage Account name:

```powershell
$STORAGE = "myaz104storage2026xxxx"
```

Remember:

> Storage Account names must be globally unique.

Create it:

```powershell
az storage account create `
    --resource-group $RG `
    --name $STORAGE `
    --location $LOCATION `
    --sku Standard_LRS
```

---

# 17. Create PublicVM

Create the VM inside `PublicSubnet`.

```powershell
az vm create `
    --resource-group $RG `
    --name PublicVM `
    --image Ubuntu2204 `
    --vnet-name $VNET `
    --subnet PublicSubnet `
    --admin-username azureuser `
    --generate-ssh-keys
```

Architecture:

```text
PublicSubnet
10.0.1.0/24
      |
      ↓
  PublicVM
```

---

# 18. Create PrivateVM

Create another VM inside `PrivateSubnet`.

```powershell
az vm create `
    --resource-group $RG `
    --name PrivateVM `
    --image Ubuntu2204 `
    --vnet-name $VNET `
    --subnet PrivateSubnet `
    --admin-username azureuser `
    --generate-ssh-keys
```

Architecture:

```text
PrivateSubnet
10.0.2.0/24
      |
      ↓
  PrivateVM
```

Now:

```text
MyVNet
 |
 +-- PublicSubnet
 |      |
 |   PublicVM
 |
 +-- PrivateSubnet
        |
     PrivateVM
```

---

# 19. Create the Private Endpoint

Now we create the Private Endpoint for the Storage Account.

First get the Storage Account resource ID:

```powershell
$STORAGE_ID = az storage account show `
    --resource-group $RG `
    --name $STORAGE `
    --query id `
    --output tsv
```

Create the Private Endpoint:

```powershell
az network private-endpoint create `
    --resource-group $RG `
    --name StoragePrivateEndpoint `
    --vnet-name $VNET `
    --subnet PrivateEndpointSubnet `
    --private-connection-resource-id $STORAGE_ID `
    --group-id blob `
    --connection-name StorageConnection
```

Notice:

```text
--subnet PrivateEndpointSubnet
```

The Private Endpoint is placed inside:

```text
PrivateEndpointSubnet
10.0.3.0/24
```

---

# 20. Find the Private Endpoint IP

Run:

```powershell
az network private-endpoint show `
    --resource-group $RG `
    --name StoragePrivateEndpoint `
    --query "customDnsConfigs[].ipAddresses[]" `
    --output tsv
```

For example:

```text
10.0.3.5
```

Your actual IP may be different.

The architecture is now:

```text
PrivateEndpointSubnet
10.0.3.0/24
        |
        ↓
Private Endpoint
        |
     10.0.3.5
        |
        ↓
Storage Account
```

---

# 21. Private DNS

The VM normally uses the Storage Account DNS name:

```text
mystorageaccount.blob.core.windows.net
```

We want that DNS name to resolve to:

```text
10.0.3.5
```

Create the Private DNS Zone:

```powershell
az network private-dns zone create `
    --resource-group $RG `
    --name privatelink.blob.core.windows.net
```

---

# 22. Link Private DNS Zone to VNet

```powershell
az network private-dns link vnet create `
    --resource-group $RG `
    --zone-name privatelink.blob.core.windows.net `
    --name MyVNetDNSLink `
    --virtual-network $VNET `
    --registration-enabled false
```

Now the DNS zone is linked to:

```text
MyVNet
```

---

# 23. Create Private DNS Zone Group

Connect the Private Endpoint to the Private DNS Zone:

```powershell
az network private-endpoint dns-zone-group create `
    --resource-group $RG `
    --endpoint-name StoragePrivateEndpoint `
    --name StorageDNSZoneGroup `
    --private-dns-zone privatelink.blob.core.windows.net `
    --zone-name blob
```

Now:

```text
VM
 |
 | mystorageaccount.blob.core.windows.net
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

---

# 24. What Happens When PublicVM Accesses Storage?

Suppose:

```text
PublicVM
10.0.1.4
```

accesses:

```text
mystorageaccount.blob.core.windows.net
```

The flow is:

```text
                         MyVNet
                           |
                     PublicSubnet
                           |
                       PublicVM
                       10.0.1.4
                           |
                           ↓
                         DNS
                           |
                           ↓
                    Private DNS Zone
                           |
                           ↓
                       10.0.3.5
                           |
                           ↓
                  Private Endpoint
                           |
                           ↓
                    Storage Account
```

The connection to Storage is through the **private endpoint**.

---

# 25. What Happens When PrivateVM Accesses Storage?

The same private endpoint can be reached from the `PrivateSubnet`.

```text
PrivateVM
10.0.2.4
     |
     ↓
Private DNS
     |
     ↓
10.0.3.5
     |
     ↓
Private Endpoint
     |
     ↓
Storage Account
```

The Private Endpoint is associated with the VNet, not with a particular VM.

Therefore both:

```text
PublicVM
PrivateVM
```

can potentially use the Private Endpoint, assuming normal network and access controls permit the connection.

---

# 26. Important: Storage Account Does Not Move Into the VNet

This is a common misunderstanding.

The Storage Account remains an Azure PaaS service.

It does **not** become:

```text
MyVNet
 |
 +-- Storage Account
```

Instead:

```text
MyVNet
 |
 +-- Private Endpoint
          |
          | Private connection
          ↓
     Storage Account
```

The Private Endpoint is the private entry point.

---

# 27. What Does the Private IP Represent?

Suppose Azure gives the Private Endpoint:

```text
10.0.3.5
```

Think of it as:

```text
10.0.3.5
     |
     ↓
Private Endpoint
     |
     ↓
Storage Account
```

The IP belongs to the Private Endpoint's network interface.

It is **not the actual IP address of the Storage Account infrastructure**.

---

# 28. Service Endpoint vs Private Endpoint

This is the most important comparison.

## Service Endpoint

```text
                    MyVNet
                       |
                 PrivateSubnet
                       |
                      VM
                       |
              Service Endpoint
               Microsoft.Storage
                       |
                       ↓
                Storage Account
```

There is **no private IP created for Storage inside your VNet**.

---

## Private Endpoint

```text
                    MyVNet
                       |
          +------------+------------+
          |                         |
     PublicSubnet        PrivateEndpointSubnet
          |                         |
      PublicVM              Private Endpoint
                                      |
                                   10.0.3.5
                                      |
                                      ↓
                               Storage Account
```

There **is a private IP**.

---

# 29. Do We Need a Service Endpoint?

**No.**

Private Endpoint and Service Endpoint are separate technologies.

You don't need to configure:

```text
Microsoft.Storage
```

as a Service Endpoint just because you are using a Storage Private Endpoint.

The private connection is:

```text
VM
 ↓
Private IP
 ↓
Private Endpoint
 ↓
Storage Account
```

---

# 30. Do We Need Private DNS?

For a practical production architecture, **Private DNS integration is commonly used**.

Without Private DNS, you could access the private IP directly:

```text
10.0.3.5
```

But applications normally use:

```text
storageaccount.blob.core.windows.net
```

Private DNS allows the normal hostname to resolve to the private endpoint IP.

```text
storageaccount.blob.core.windows.net
             |
             ↓
        Private DNS
             |
             ↓
          10.0.3.5
             |
             ↓
      Private Endpoint
```

---

# 31. Does Private Endpoint Automatically Make Everything Private?

No.

Creating a Private Endpoint provides a private path to the Azure service.

You should also consider the PaaS service's public network access configuration.

For example, with Storage Account networking you can restrict or disable public network access according to your architecture.

Conceptually:

```text
Internet
   |
   X
Public access restricted
   |
Storage Account
   ↑
   |
Private Endpoint
   ↑
   |
MyVNet
```

This is a common architecture when you want application traffic to use private connectivity.

---

# 32. Our Three Subnets

### PublicSubnet

```text
10.0.1.0/24
     |
     ↓
 PublicVM
```

### PrivateSubnet

```text
10.0.2.0/24
     |
     ↓
 PrivateVM
```

### PrivateEndpointSubnet

```text
10.0.3.0/24
     |
     ↓
Private Endpoint
     |
  10.0.3.5
     |
     ↓
Storage Account
```

---

# 33. Final Architecture

This is the main diagram to remember:

```text
                           Azure
                             |
                      MYRG-India
                             |
                           MyVNet
                        10.0.0.0/16
                             |
          +------------------+------------------+
          |                  |                  |
          ↓                  ↓                  ↓
    PublicSubnet       PrivateSubnet     PrivateEndpointSubnet
     10.0.1.0/24        10.0.2.0/24          10.0.3.0/24
          |                  |                  |
          ↓                  ↓                  ↓
      PublicVM           PrivateVM        Private Endpoint
      10.0.1.4           10.0.2.4              |
                                                |
                                             10.0.3.5
                                                |
                                                ↓
                                         Storage Account
```

Access:

```text
PublicVM / PrivateVM
          |
          ↓
      Private DNS
          |
          ↓
       10.0.3.5
          |
          ↓
   Private Endpoint
          |
          ↓
   Storage Account
```

---

# 34. AZ-104 Memory Trick

Remember these two phrases.

## Service Endpoint

> **SUBNET → SERVICE**

```text
Subnet
   ↓
Service Endpoint
   ↓
Azure Service
```

## Private Endpoint

> **PRIVATE IP → SERVICE**

```text
Private IP
   ↓
Private Endpoint
   ↓
Azure Service
```

The shortest memory trick:

```text
SERVICE ENDPOINT
       ↓
    SUBNET

PRIVATE ENDPOINT
       ↓
   PRIVATE IP
```

---

# 35. One-Line Definitions

### Service Endpoint

> **A subnet-level feature that enables access to supported Azure PaaS services and allows the service to identify traffic originating from the VNet/subnet.**

### Private Endpoint

> **A network interface that assigns a private IP address from your VNet to provide private connectivity to an Azure PaaS service.**

### Private DNS

> **Private DNS allows the normal service hostname to resolve to the private IP address of the Private Endpoint.**

---

# 36. Final Comparison

| Feature                     | Service Endpoint     | Private Endpoint         |
| --------------------------- | -------------------- | ------------------------ |
| Configured at               | Subnet               | Subnet                   |
| Private IP for PaaS service | ❌ No                 | ✅ Yes                    |
| Private IP in VNet          | ❌                    | ✅                        |
| Private DNS commonly used   | Not required         | Commonly used            |
| Main concept                | Subnet-based access  | Private connectivity     |
| Example                     | Subnet → Storage     | Private IP → Storage     |
| Memory trick                | **SUBNET → SERVICE** | **PRIVATE IP → SERVICE** |

---

# 37. Final AZ-104 Picture

### Service Endpoint

```text
VM
 |
Subnet
 |
Service Endpoint
 |
Storage
```

### Private Endpoint

```text
VM
 |
VNet
 |
Private IP
 |
Private Endpoint
 |
Storage
```

## The easiest way to remember

> **Service Endpoint tells the Azure service: "This request is coming from this subnet."**

> **Private Endpoint gives your VNet: "A private IP through which you can reach the Azure service."**
