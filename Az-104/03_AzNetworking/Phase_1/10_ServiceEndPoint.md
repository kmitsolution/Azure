# Azure Service Endpoint

## 1. What is a Service Endpoint?

An **Azure Service Endpoint** allows a subnet in your Azure VNet to connect to supported Azure PaaS services, such as **Storage Account, Azure SQL, Key Vault**, etc., using Azure's backbone network.

The most important point is:

> **Service Endpoints are configured at the SUBNET level.**

Not at the VM level.

Think of it like this:

```text
VNet
│
├── PublicSubnet
│
└── PrivateSubnet
       │
       ├── VM1
       └── Service Endpoint
              │
              │ Microsoft.Storage
              ↓
        Azure Storage Account
```

The VM doesn't individually get a Service Endpoint.

Instead:

```text
Subnet → Service Endpoint → Azure Service
```

Therefore, **any resource in that subnet can use the configured Service Endpoint**, subject to the destination service's network access rules.

---

# 2. Why do we need Service Endpoints?

Suppose you have a VM:

```text
VM
10.0.2.4
```

inside:

```text
PrivateSubnet
10.0.2.0/24
```

And your application on the VM needs to access an Azure Storage Account.

Without a Service Endpoint, you can access the Storage Account through its normal service endpoint.

Conceptually:

```text
VM
 |
 |
 v
Azure Storage
```

But suppose your requirement is:

> "I don't want my Storage Account to be accessible from anywhere. I want to allow access from my specific Azure subnet."

This is where Service Endpoint becomes useful.

You configure:

```text
PrivateSubnet
      |
      +---- Microsoft.Storage
```

Then configure the Storage Account's networking rules to allow that subnet.

Now:

```text
PrivateSubnet
      |
      | Microsoft.Storage
      ↓
Storage Account
      |
      +-- Allow PrivateSubnet
      +-- Deny other networks
```

---

# 3. Real-world use case

Imagine a 3-tier application:

```text
                 Azure VNet
              10.0.0.0/16
                    |
        +-----------+-----------+
        |                       |
   WebSubnet              AppSubnet
        |                       |
      Web VM                 App VM
                                |
                                |
                         needs Storage
                                |
                                ↓
                         Storage Account
```

You don't want every network to access your Storage Account.

You want:

```text
AppSubnet → Storage = ALLOW

Internet → Storage = DENY
OtherSubnet → Storage = DENY
```

So you enable:

```text
Microsoft.Storage
```

on `AppSubnet`.

Then tell the Storage Account:

```text
Allow AppSubnet
```

This is the primary use case.

---

# 4. Very important: Service Endpoint is configured on SUBNET

This is the point I would emphasize in your AZ-104 class.

Suppose:

```text
MyVNet
│
├── PublicSubnet
│      └── VM1
│
└── PrivateSubnet
       ├── VM2
       └── VM3
```

You configure:

```text
PrivateSubnet
      |
      +-- Microsoft.Storage
```

Then:

```text
VM2 ──┐
      ├──> Storage
VM3 ──┘
```

Both VM2 and VM3 can use the Service Endpoint because **both are inside the subnet**.

But VM1 is in `PublicSubnet`.

Therefore:

```text
VM1 → Storage
```

doesn't automatically get the Service Endpoint because its subnet doesn't have:

```text
Microsoft.Storage
```

enabled.

---

# 5. What actually happens when VM accesses Storage?

Let's follow the request.

Suppose:

```text
VNet
10.0.0.0/16

PrivateSubnet
10.0.2.0/24

VM
10.0.2.4
```

Storage:

```text
mystorageaccount.blob.core.windows.net
```

The application on the VM says:

```text
"I want to access Blob Storage."
```

The request goes through Azure networking.

Because the subnet has:

```text
Microsoft.Storage
```

Service Endpoint enabled, Azure can identify the originating subnet/VNet to the Storage service.

Conceptually:

```text
                  Azure VNet
                      |
                      |
               PrivateSubnet
                10.0.2.0/24
                      |
                      |
                     VM
                  10.0.2.4
                      |
                      |
              Service Endpoint
              Microsoft.Storage
                      |
                      |
                      ↓
               Azure Storage
                      |
                      |
               Network Rules
                      |
                AppSubnet?
                 /       \
               YES        NO
                |          |
              ALLOW       DENY
```

The **Storage Account's network rules** determine whether that subnet is allowed.

---

# 6. Service Endpoint does NOT mean Private IP

This is another very important distinction.

With Service Endpoint:

```text
VM
 |
 |
Service Endpoint
 |
 |
Storage
```

You **do not get a private IP address for the Storage Account inside your VNet**.

That's different from a **Private Endpoint**.

### Service Endpoint

```text
Subnet
   |
   ↓
Azure Storage
```

### Private Endpoint

```text
VNet
 |
 |
Private Endpoint
 |
 +---- Private IP: 10.0.2.x
             |
             ↓
        Storage Account
```

So remember:

> **Service Endpoint = subnet-based access**

> **Private Endpoint = private IP**

---

# 7. Azure Portal — How do we configure it?

Let's use this example:

```text
Resource Group: MYRG-India

VNet:
MyVNet

Subnet:
PrivateSubnet

VM:
PrivateVM

Storage:
myaz104storagexxxx
```

## Step 1 — Open VNet

Azure Portal:

```text
Azure Portal
   ↓
Virtual Networks
   ↓
MyVNet
```

Go to:

```text
Subnets
```

You will see:

```text
PublicSubnet
PrivateSubnet
```

Click:

```text
PrivateSubnet
```

---

# 8. Enable Service Endpoint

Inside the subnet configuration, find:

```text
Service endpoints
```

Select:

```text
Microsoft.Storage
```

So it becomes:

```text
PrivateSubnet

Service endpoints:
    Microsoft.Storage
```

Click:

```text
Save
```

That's it.

The Service Endpoint is now configured on the subnet.

---

# 9. Configure Storage Account

Now go to:

```text
Storage Account
   ↓
Networking
```

Choose the appropriate network access option, such as:

```text
Selected networks
```

Then add:

```text
Virtual networks
```

Select:

```text
MyVNet
   ↓
PrivateSubnet
```

Save.

Now the architecture is:

```text
             MyVNet
               |
        PrivateSubnet
               |
          Service Endpoint
         Microsoft.Storage
               |
               ↓
        Storage Account
               |
       Network restriction
               |
        PrivateSubnet
           ALLOW
```

---

# 10. Azure CLI — Enable Service Endpoint

Now let's do exactly the same thing using Azure CLI.

Assume:

```text
Resource Group = MYRG-India
VNet = MyVNet
Subnet = PrivateSubnet
```

### Windows PowerShell

Since you're using Windows:

```powershell
az network vnet subnet update `
    --resource-group MYRG-India `
    --vnet-name MyVNet `
    --name PrivateSubnet `
    --service-endpoints Microsoft.Storage
```

### Linux

```bash
az network vnet subnet update \
    --resource-group MYRG-India \
    --vnet-name MyVNet \
    --name PrivateSubnet \
    --service-endpoints Microsoft.Storage
```

Notice something very important:

```text
az network vnet subnet update
```

We are updating the **SUBNET**.

Not:

```text
az vm update
```

Not:

```text
az storage account update
```

The Service Endpoint belongs to the subnet configuration.

---

# 11. Verify Service Endpoint

Windows PowerShell:

```powershell
az network vnet subnet show `
    --resource-group MYRG-India `
    --vnet-name MyVNet `
    --name PrivateSubnet `
    --query serviceEndpoints
```

You should see something like:

```text
[
  {
    "locations": [
      "centralindia"
    ],
    "provisioningState": "Succeeded",
    "service": "Microsoft.Storage"
  }
]
```

You can also use:

```powershell
az network vnet subnet show `
    --resource-group MYRG-India `
    --vnet-name MyVNet `
    --name PrivateSubnet `
    --query "serviceEndpoints[].service"
```

Result:

```text
Microsoft.Storage
```

---

# 12. Create Storage Account using CLI

For example:

```powershell
az storage account create `
    --resource-group MYRG-India `
    --name myaz104storage12345 `
    --location centralindia `
    --sku Standard_LRS
```

Remember:

**Storage Account names must be globally unique.**

---

# 13. Allow the subnet in Storage Account

Now configure the Storage Account network rules:

```powershell
az storage account network-rule add `
    --resource-group MYRG-India `
    --account-name myaz104storage12345 `
    --vnet-name MyVNet `
    --subnet PrivateSubnet
```

Then set the default network action to deny:

```powershell
az storage account update `
    --resource-group MYRG-India `
    --name myaz104storage12345 `
    --default-action Deny
```

Now the important relationship is:

```text
PrivateSubnet
      |
      | Service Endpoint
      | Microsoft.Storage
      ↓
Storage Account
      |
      | Network Rule
      ↓
PrivateSubnet = ALLOW
Everything else = DENY
```

---

# 14. What is the difference between these two commands?

This is an excellent AZ-104 exam concept.

### Command 1

```powershell
az network vnet subnet update `
    --service-endpoints Microsoft.Storage
```

This says:

> "Enable Storage Service Endpoint on this subnet."

### Command 2

```powershell
az storage account network-rule add `
    --vnet-name MyVNet `
    --subnet PrivateSubnet
```

This says:

> "Allow this VNet/subnet to access the Storage Account."

They work together.

---

# 15. Complete Console Architecture

Your final architecture looks like this:

```text
                    Azure
                      |
                Resource Group
                  MYRG-India
                      |
                    MyVNet
                 10.0.0.0/16
                      |
             +--------+--------+
             |                 |
       PublicSubnet      PrivateSubnet
                          10.0.2.0/24
                               |
                               |
                              VM
                               |
                               |
                     Service Endpoint
                      Microsoft.Storage
                               |
                               |
                               ↓
                       Storage Account
                               |
                       Network Rules
                               |
                      PrivateSubnet
                           ALLOW
```

---

# 16. What if another VM is in another subnet?

Suppose:

```text
MyVNet
│
├── PublicSubnet
│      └── VM1
│
└── PrivateSubnet
       └── VM2
```

Only:

```text
PrivateSubnet
      |
      +-- Microsoft.Storage
```

has the Service Endpoint.

Therefore:

```text
VM2 → Storage
```

can use the Service Endpoint.

But:

```text
VM1 → Storage
```

doesn't get that Service Endpoint because VM1 is in `PublicSubnet`.

This demonstrates why we say:

> **Service Endpoint is a subnet-level configuration.**

---

# 17. What problem does Service Endpoint solve?

Without Service Endpoint:

```text
How do I restrict Storage access
to a particular Azure subnet?
```

With Service Endpoint:

```text
PrivateSubnet
      |
      +-- Microsoft.Storage
      |
      ↓
Storage Account
      |
      +-- Allow PrivateSubnet
```

So the main use case is:

> **Restrict access to supported Azure PaaS services based on the source VNet/subnet.**

---

# 18. Common Service Endpoint Services

Some commonly encountered service endpoints include:

```text
Microsoft.Storage
Microsoft.Sql
Microsoft.KeyVault
Microsoft.AzureCosmosDB
Microsoft.ServiceBus
Microsoft.EventHub
Microsoft.Web
```

For example:

```text
PrivateSubnet
      |
      +-- Microsoft.Storage
      +-- Microsoft.Sql
      +-- Microsoft.KeyVault
```

A subnet can have multiple service endpoints.

---

# 19. Service Endpoint vs Private Endpoint

This is probably the most important comparison to remember for AZ-104.

| Service Endpoint                            | Private Endpoint                     |
| ------------------------------------------- | ------------------------------------ |
| Configured on subnet                        | Creates a private endpoint resource  |
| Subnet-level feature                        | Gets a private IP in VNet            |
| Access to supported PaaS service            | Private connectivity to PaaS service |
| PaaS service still has its service endpoint | Uses private IP                      |
| Simple configuration                        | More private/isolation-oriented      |

Think:

```text
SERVICE ENDPOINT
       ↓
"Which SUBNET is accessing my service?"
```

versus:

```text
PRIVATE ENDPOINT
       ↓
"Give my service a PRIVATE IP inside my VNet."
```

---

# 20. The Best Memory Trick

I recommend remembering this:

## **SE = Subnet → Service**

**S**ervice **E**ndpoint:

```text
S → S
Service Endpoint → Subnet → Service
```

Or even simpler:

> **Service Endpoint starts at the SUBNET and points toward a SERVICE.**

```text
SUBNET
   |
   | Service Endpoint
   ↓
SERVICE
```

For your Storage example:

```text
PrivateSubnet
      |
      | Microsoft.Storage
      ↓
Storage Account
```

---

# 21. Three Things to Remember for AZ-104

If you remember only three things, remember these:

### ① Service Endpoint is configured on the SUBNET

```text
VNet
 ↓
Subnet
 ↓
Service Endpoint
```

### ② It is used to access supported Azure PaaS services

```text
Subnet
 ↓
Microsoft.Storage
 ↓
Storage
```

### ③ It is NOT a Private Endpoint

```text
Service Endpoint
→ subnet-based access
→ no private IP for the PaaS service
```

```text
Private Endpoint
→ private IP
→ private connectivity
```

## One-line definition for your notes

> **An Azure Service Endpoint is a subnet-level configuration that enables a subnet to access supported Azure PaaS services over the Azure backbone and allows the PaaS service to restrict access based on the originating VNet/subnet.**

### The picture to remember

```text
                  VNet
                   |
             ┌─────┴─────┐
             |           |
        PublicSubnet  PrivateSubnet
                          |
                         VM
                          |
                 Service Endpoint
                  Microsoft.Storage
                          |
                          ↓
                   Storage Account
                          |
                   Network Rules
                          |
                  PrivateSubnet
                     ALLOW
```

**Memory trick: `SUBNET → SERVICE` = Service Endpoint.**
