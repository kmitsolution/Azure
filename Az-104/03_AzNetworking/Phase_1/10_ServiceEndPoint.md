## Azure Service Endpoint 

### Objective

In this lab, we will understand and configure an **Azure Service Endpoint**.

We will:

* Create a VNet
* Create a subnet
* Create a Storage Account
* Enable the `Microsoft.Storage` Service Endpoint on the subnet
* Configure Storage Account network access
* Allow the subnet to access the Storage Account
* Deny access from other networks
* Test the configuration

---

# 1. What is a Service Endpoint?

A **Service Endpoint** allows a subnet in an Azure VNet to connect to supported Azure PaaS services using the Azure backbone network.

For example:

```text
                    Azure VNet
                 10.0.0.0/16
                       |
                       |
                 PrivateSubnet
                  10.0.2.0/24
                       |
                       |
                       VM
                       |
                       |
             Microsoft.Storage
             Service Endpoint
                       |
                       |
                       v
                Storage Account
```

The Service Endpoint allows the Azure service to identify traffic coming from the configured VNet/subnet.

---

# 2. Service Endpoint Does NOT Create a Private IP

This is one of the most important concepts.

With a Service Endpoint:

```text
VM
 |
 |
 +------ Azure backbone ------+
                              |
                              v
                       Storage Account
                       Public Endpoint
```

The Storage Account still has its normal Azure service endpoint.

A Service Endpoint does **not** create a private IP address for the Storage Account.

---

# 3. Service Endpoint vs Private Endpoint

| Service Endpoint                 | Private Endpoint                     |
| -------------------------------- | ------------------------------------ |
| Configured on subnet             | Creates private endpoint/NIC         |
| Uses service's existing endpoint | Provides private IP                  |
| Service can identify subnet      | Service accessed through private IP  |
| No private IP for Storage        | Private IP in VNet                   |
| Simple configuration             | More private/network-isolated design |

### Memory Trick

```text
Service Endpoint
      ↓
"Allow this subnet to access the service"

Private Endpoint
      ↓
"Give the service a private IP in my VNet"
```

---

# 4. Lab Architecture

We will create:

```text
Resource Group
    |
    +-- MyVNet
         |
         +-- PrivateSubnet
              |
              +-- Microsoft.Storage
                    |
                    v
              Storage Account
```

Example:

```text
Resource Group: MYRG-India

VNet:
MyVNet
10.0.0.0/16

Subnet:
PrivateSubnet
10.0.2.0/24

Service Endpoint:
Microsoft.Storage

Storage Account:
myaz104storageXXXX
```

---

# 5. Azure CLI Syntax

The Azure CLI command itself is the same on Linux and Windows.

The difference is line continuation.

### Linux / macOS

Use:

```text
\
```

Example:

```bash
az network vnet create \
  --resource-group MYRG-India \
  --name MyVNet \
  --location centralindia \
  --address-prefixes 10.0.0.0/16
```

### Windows PowerShell

Use:

```text
`
```

Example:

```powershell
az network vnet create `
  --resource-group MYRG-India `
  --name MyVNet `
  --location centralindia `
  --address-prefixes 10.0.0.0/16
```

---

# 6. Login to Azure

## Linux / macOS

```bash
az login
```

## Windows PowerShell

```powershell
az login
```

Check the current subscription:

```bash
az account show --output table
```

or:

```powershell
az account show --output table
```

If you have multiple subscriptions:

```bash
az account list --output table
```

Set the required subscription:

```bash
az account set --subscription "<SUBSCRIPTION-NAME-OR-ID>"
```

---

# 7. Set Variables

## Linux / macOS

```bash
RG="MYRG-India"
LOCATION="centralindia"
VNET="MyVNet"
```

## Windows PowerShell

```powershell
$RG = "MYRG-India"
$LOCATION = "centralindia"
$VNET = "MyVNet"
```

---

# 8. Verify Resource Group

The resource group already exists.

## Linux / macOS

```bash
az group show \
  --name $RG \
  --output table
```

## Windows PowerShell

```powershell
az group show `
  --name $RG `
  --output table
```

---

# 9. Create VNet

Create:

```text
VNet Name: MyVNet
Address Space: 10.0.0.0/16
```

## Linux / macOS

```bash
az network vnet create \
  --resource-group $RG \
  --name $VNET \
  --location $LOCATION \
  --address-prefixes 10.0.0.0/16
```

## Windows PowerShell

```powershell
az network vnet create `
  --resource-group $RG `
  --name $VNET `
  --location $LOCATION `
  --address-prefixes 10.0.0.0/16
```

Verify:

### Linux / macOS

```bash
az network vnet show \
  --resource-group $RG \
  --name $VNET \
  --output table
```

### Windows PowerShell

```powershell
az network vnet show `
  --resource-group $RG `
  --name $VNET `
  --output table
```

---

# 10. Create PrivateSubnet

Create:

```text
PrivateSubnet
10.0.2.0/24
```

## Linux / macOS

```bash
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --address-prefixes 10.0.2.0/24
```

## Windows PowerShell

```powershell
az network vnet subnet create `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --address-prefixes 10.0.2.0/24
```

Verify:

### Linux / macOS

```bash
az network vnet subnet list \
  --resource-group $RG \
  --vnet-name $VNET \
  --output table
```

### Windows PowerShell

```powershell
az network vnet subnet list `
  --resource-group $RG `
  --vnet-name $VNET `
  --output table
```

---

# 11. Create Storage Account

Storage Account names must be globally unique.

For example:

```text
myaz104storage2026xxxx
```

Replace the name with your own unique name.

## Linux / macOS

```bash
STORAGE="myaz104storage2026xxxx"

az storage account create \
  --resource-group $RG \
  --name $STORAGE \
  --location $LOCATION \
  --sku Standard_LRS
```

## Windows PowerShell

```powershell
$STORAGE = "myaz104storage2026xxxx"

az storage account create `
  --resource-group $RG `
  --name $STORAGE `
  --location $LOCATION `
  --sku Standard_LRS
```

Verify:

### Linux / macOS

```bash
az storage account show \
  --resource-group $RG \
  --name $STORAGE \
  --output table
```

### Windows PowerShell

```powershell
az storage account show `
  --resource-group $RG `
  --name $STORAGE `
  --output table
```

---

# 12. Enable Service Endpoint on PrivateSubnet

This is the most important step.

We will enable:

```text
Microsoft.Storage
```

on:

```text
PrivateSubnet
```

## Linux / macOS

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --service-endpoints Microsoft.Storage
```

## Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --service-endpoints Microsoft.Storage
```

Now the architecture becomes:

```text
MyVNet
 |
 +-- PrivateSubnet
       |
       +-- Service Endpoint
              |
              +-- Microsoft.Storage
```

---

# 13. Verify Service Endpoint

## Linux / macOS

```bash
az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --query serviceEndpoints \
  --output table
```

## Windows PowerShell

```powershell
az network vnet subnet show `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --query serviceEndpoints `
  --output table
```

Expected:

```text
Service
----------------
Microsoft.Storage
```

---

# 14. Configure Storage Account Network Access

By default, a Storage Account can have public network access enabled.

We will configure it to allow only selected networks.

First, add our subnet as an allowed virtual network.

## Linux / macOS

```bash
az storage account network-rule add \
  --resource-group $RG \
  --account-name $STORAGE \
  --vnet-name $VNET \
  --subnet PrivateSubnet
```

## Windows PowerShell

```powershell
az storage account network-rule add `
  --resource-group $RG `
  --account-name $STORAGE `
  --vnet-name $VNET `
  --subnet PrivateSubnet
```

---

# 15. Change Storage Default Network Action to Deny

Now configure the Storage Account so that network traffic is denied by default.

## Linux / macOS

```bash
az storage account update \
  --resource-group $RG \
  --name $STORAGE \
  --default-action Deny
```

## Windows PowerShell

```powershell
az storage account update `
  --resource-group $RG `
  --name $STORAGE `
  --default-action Deny
```

Now the model is:

```text
                     Storage Account
                           |
                    Network Firewall
                           |
                 +---------+---------+
                 |                   |
        PrivateSubnet            Other Networks
                 |                   |
               ALLOW                DENY
```

---

# 16. Verify Storage Network Rules

## Linux / macOS

```bash
az storage account show \
  --resource-group $RG \
  --name $STORAGE \
  --query networkRuleSet
```

## Windows PowerShell

```powershell
az storage account show `
  --resource-group $RG `
  --name $STORAGE `
  --query networkRuleSet
```

You should see information about:

```text
defaultAction
virtualNetworkRules
```

The default action should be:

```text
Deny
```

and `PrivateSubnet` should appear in the virtual network rules.

---

# 17. Final Architecture

```text
                         Azure
                           |
                           |
                    Resource Group
                     MYRG-India
                           |
                           |
                        MyVNet
                     10.0.0.0/16
                           |
                           |
                    PrivateSubnet
                     10.0.2.0/24
                           |
                           |
                 Service Endpoint
                  Microsoft.Storage
                           |
                           |
                           v
                    Storage Account
                           |
                    Storage Firewall
                           |
              +------------+------------+
              |                         |
              |                         |
        PrivateSubnet              Other Networks
           ALLOW                       DENY
```

---

# 18. What Happens When the VM Accesses Storage?

Suppose a VM exists in:

```text
PrivateSubnet
10.0.2.0/24
```

The VM accesses:

```text
Storage Account
```

The flow is:

```text
VM
 |
 |
 | Azure networking
 |
 v
Service Endpoint
Microsoft.Storage
 |
 |
 v
Storage Account
 |
 |
 v
Storage Network Rules
 |
 +-- PrivateSubnet = ALLOW
 |
 v
Access Granted
```

The Service Endpoint allows the Storage service to recognize the traffic as coming from the configured subnet.

---

# 19. What Happens From Another Network?

Suppose another VM is located in:

```text
OtherSubnet
10.0.3.0/24
```

and that subnet does not have the appropriate Service Endpoint/network rule.

The request reaches the Storage Account:

```text
Other VM
    |
    v
Storage Account
    |
    v
Storage Firewall
    |
    +-- Not an allowed subnet
    |
    v
DENY
```

Therefore:

```text
PrivateSubnet → Storage = ALLOW

OtherSubnet → Storage = DENY
```

---

# 20. Add Multiple Service Endpoints

A subnet can have multiple service endpoints.

For example:

```text
PrivateSubnet
 |
 +-- Microsoft.Storage
 |
 +-- Microsoft.Sql
 |
 +-- Microsoft.KeyVault
```

## Linux / macOS

You can specify multiple services:

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --service-endpoints Microsoft.Storage Microsoft.Sql Microsoft.KeyVault
```

## Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --service-endpoints Microsoft.Storage Microsoft.Sql Microsoft.KeyVault
```

Verify:

```bash
az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --query serviceEndpoints \
  --output table
```

---

# 21. Service Endpoint for Azure SQL

For example, enable:

```text
Microsoft.Sql
```

on a subnet.

## Linux / macOS

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --service-endpoints Microsoft.Sql
```

## Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --service-endpoints Microsoft.Sql
```

The SQL service can then use virtual network rules to restrict access to the selected subnet.

---

# 22. Service Endpoint for Key Vault

You can also enable:

```text
Microsoft.KeyVault
```

## Linux / macOS

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --service-endpoints Microsoft.KeyVault
```

## Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --service-endpoints Microsoft.KeyVault
```

---

# 23. Remove a Service Endpoint

If you want to remove `Microsoft.Storage` from the subnet, you can update the subnet's service endpoint configuration.

First, check the current endpoints:

```bash
az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --query serviceEndpoints
```

When removing an endpoint, make sure you preserve any other service endpoints that you still need.

For example, if Storage is the only endpoint:

## Linux / macOS

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --service-endpoints
```

## Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --service-endpoints
```

---

# 24. Service Endpoint vs Internet Access

A Service Endpoint does not mean:

```text
VM → Internet → Storage
```

Conceptually:

```text
VM
 |
 |
Azure Network
 |
 |
Azure Backbone
 |
 |
Storage Service
```

The traffic uses Microsoft's Azure network rather than requiring the application to traverse the public internet path.

---

# 25. Key Points for AZ-104

### Service Endpoint

> A Service Endpoint enables a subnet to access supported Azure PaaS services and allows the service to restrict access based on the originating VNet/subnet.

### Service Endpoint is configured on:

```text
Subnet
```

Not directly on:

```text
VM
Storage Account
```

The configuration is:

```text
VNet
 |
 +-- Subnet
       |
       +-- Service Endpoint
```

---

# 26. Service Endpoint Does Not Give a Private IP

Remember:

```text
Service Endpoint
       ↓
No private IP created for PaaS service
```

Whereas:

```text
Private Endpoint
       ↓
Private IP created in your VNet
```

---

# 27. Service Endpoint + Storage Firewall

The two work together:

```text
Subnet
  |
  +-- Microsoft.Storage Service Endpoint
  |
  v
Storage Account
  |
  +-- Network Rules
       |
       +-- Allow PrivateSubnet
       |
       +-- Deny Others
```

The Service Endpoint alone does **not** automatically mean the Storage Account will allow the subnet.

You normally also configure the Storage Account's network access rules.

---

# 28. AZ-104 Memory Trick

```text
Service Endpoint
       =
Subnet → Azure PaaS Service
```

```text
Private Endpoint
       =
Private IP → Azure PaaS Service
```

```text
Storage Firewall
       =
Which networks can access Storage?
```

### Simple comparison

```text
Service Endpoint
"Identify my subnet"

Storage Firewall
"Allow my subnet"

Private Endpoint
"Give me a private IP"
```

---

# 29. Complete Lab Flow

```text
1. Create Resource Group
          ↓
2. Create VNet
          ↓
3. Create Subnet
          ↓
4. Create Storage Account
          ↓
5. Enable Microsoft.Storage
   Service Endpoint on Subnet
          ↓
6. Add Subnet to Storage
   Network Rules
          ↓
7. Set Storage Default Action = Deny
          ↓
8. Test access
```

---

# 30. Final AZ-104 Definition

> **Azure Service Endpoint provides secure, direct connectivity from a VNet subnet to supported Azure PaaS services over the Azure backbone and allows the PaaS service to restrict access based on the originating subnet.**

```text
             SERVICE ENDPOINT

                  VNet
                   |
                 Subnet
                   |
          Microsoft.Storage
                   |
                   v
            Storage Account
                   |
            Network Rules
                   |
             Allow / Deny
```

**Key exam point:**

```text
Service Endpoint → configured on SUBNET

Private Endpoint → creates PRIVATE IP

Storage Network Rules → control which networks are allowed
```
