8
 **Azure CLI commands for both Linux/macOS Bash and Windows PowerShell**. There are **no Azure PowerShell (`Az`) cmdlets**—only Azure CLI.

# AZ-104 Hands-On Lab

## Create VNet, Subnets, NSGs and VMs Using Azure CLI

### Objective

In this lab, we will create an Azure network and two Ubuntu VMs using **Azure CLI**.

We will demonstrate:

* Creating a VNet
* Creating two subnets
* Creating NSGs
* Creating inbound NSG rules
* Associating an NSG with a subnet
* Creating VMs in specific subnets
* Finding VM NIC IDs
* Creating an NSG for NICs
* Associating an NSG with NICs
* Understanding subnet-level and NIC-level NSGs
* Removing an NSG from a NIC
* Removing an NSG from a subnet

> **Important:** The Azure CLI commands are the same on Linux and Windows. Only the line-continuation syntax is different.

---

# 1. Lab Architecture

We will use the existing resource group:

```text
Resource Group: MYRG-India
```

Create the following VNet:

```text
VNet: MyVNet
Address Space: 10.0.0.0/16
```

Create two subnets:

```text
PublicSubnet
10.0.10.0/24

PrivateSubnet
10.0.2.0/24
```

Final architecture:

```text
                         MYRG-India
                              |
                           MyVNet
                        10.0.0.0/16
                              |
                 +------------+------------+
                 |                         |
                 |                         |
          PublicSubnet              PrivateSubnet
          10.0.10.0/24              10.0.2.0/24
                 |                         |
                 |                         |
       PublicSubnetNSG            PrivateSubnetNSG
                 |                         |
          Allow TCP 22              Allow TCP 22
                                    Allow TCP 80
                 |                         |
                 |                         |
             PublicVM                  PrivateVM
                 |                         |
            PublicVM-nic           PrivateVM-nic
                 |                         |
                 +------------+------------+
                              |
                           NICNSG
                              |
                       Deny TCP 22
```

---

# 2. Azure CLI Syntax: Linux vs Windows

## Linux / macOS

Linux uses `\` for line continuation:

```bash
az network vnet create \
  --resource-group MYRG-India \
  --name MyVNet \
  --location centralindia \
  --address-prefixes 10.0.0.0/16
```

## Windows PowerShell

Windows PowerShell uses the backtick `` ` ``:

```powershell
az network vnet create `
  --resource-group MYRG-India `
  --name MyVNet `
  --location centralindia `
  --address-prefixes 10.0.0.0/16
```

## Windows Command Prompt

CMD uses `^`:

```cmd
az network vnet create ^
  --resource-group MYRG-India ^
  --name MyVNet ^
  --location centralindia ^
  --address-prefixes 10.0.0.0/16
```

For this lab, we will show **Linux/macOS Bash** and **Windows PowerShell**.

---

# 3. Login to Azure

## Linux / macOS

```bash
az login
```

## Windows PowerShell

```powershell
az login --use-device
```

Check the current subscription:

```bash
az account show
```

or:

```powershell
az account show
```

List subscriptions:

```bash
az account list --output table
```

Select the required subscription:

```bash
az account set --subscription "<SUBSCRIPTION-NAME-OR-ID>"
```

Verify:

```bash
az account show --output table
```

---

# 4. Define Variables

Using variables makes the commands easier to reuse.

## Linux / macOS

```bash
RG="MYRG-India"
LOCATION="centralindia"
VNET="MyVNet"
```

Verify:

```bash
echo $RG
echo $LOCATION
echo $VNET
```

## Windows PowerShell

```powershell
$RG = "MYRG-India"
$LOCATION = "centralindia"
$VNET = "MyVNet"
```

Verify:

```powershell
$RG
$LOCATION
$VNET
```

---

# 5. Verify Resource Group

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

If the resource group does not exist, you can create it:

### Linux / macOS

```bash
az group create \
  --name $RG \
  --location $LOCATION
```

### Windows PowerShell

```powershell
az group create `
  --name $RG `
  --location $LOCATION
```

---

# 6. Create VNet

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
  --query "{Name:name,AddressSpace:addressSpace.addressPrefixes}" \
  --output json
```

### Windows PowerShell

```powershell
az network vnet show `
  --resource-group $RG `
  --name $VNET `
  --query "{Name:name,AddressSpace:addressSpace.addressPrefixes}" `
  --output json
```

Expected:

```text
MyVNet
10.0.0.0/16
```

---

# 7. Create PublicSubnet

Create:

```text
Name: PublicSubnet
CIDR: 10.0.10.0/24
```

> **Note:** The correct CIDR is `10.0.10.0/24`, not `10.0.10/24`.

## Linux / macOS

```bash
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PublicSubnet \
  --address-prefixes 10.0.10.0/24
```

## Windows PowerShell

```powershell
az network vnet subnet create `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PublicSubnet `
  --address-prefixes 10.0.10.0/24
```

---

# 8. Create PrivateSubnet

Create:

```text
Name: PrivateSubnet
CIDR: 10.0.2.0/24
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

List the subnets:

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

Expected:

```text
Name             Address Prefix
---------------  --------------
PublicSubnet     10.0.10.0/24
PrivateSubnet    10.0.2.0/24
```

---

# 9. Create PublicSubnetNSG

Create:

```text
NSG: PublicSubnetNSG
```

## Linux / macOS

```bash
az network nsg create \
  --resource-group $RG \
  --name PublicSubnetNSG \
  --location $LOCATION
```

## Windows PowerShell

```powershell
az network nsg create `
  --resource-group $RG `
  --name PublicSubnetNSG `
  --location $LOCATION
```

---

# 10. Create SSH Allow Rule on PublicSubnetNSG

Requirement:

```text
Direction: Inbound
Protocol: TCP
Port: 22
Source: Any
Action: Allow
Priority: 100
```

## Linux / macOS

```bash
az network nsg rule create \
  --resource-group $RG \
  --nsg-name PublicSubnetNSG \
  --name Allow-SSH \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22
```

## Windows PowerShell

```powershell
az network nsg rule create `
  --resource-group $RG `
  --nsg-name PublicSubnetNSG `
  --name Allow-SSH `
  --priority 100 `
  --direction Inbound `
  --access Allow `
  --protocol Tcp `
  --source-address-prefixes "*" `
  --destination-address-prefixes "*" `
  --destination-port-ranges 22
```

Verify:

### Linux / macOS

```bash
az network nsg rule list \
  --resource-group $RG \
  --nsg-name PublicSubnetNSG \
  --output table
```

### Windows PowerShell

```powershell
az network nsg rule list `
  --resource-group $RG `
  --nsg-name PublicSubnetNSG `
  --output table
```

---

# 11. Attach PublicSubnetNSG to PublicSubnet

This is a **subnet-level NSG association**.

```text
MyVNet
   |
   +-- PublicSubnet
          |
          +-- PublicSubnetNSG
```

## Linux / macOS

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PublicSubnet \
  --network-security-group PublicSubnetNSG
```

## Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PublicSubnet `
  --network-security-group PublicSubnetNSG
```

Verify:

### Linux / macOS

```bash
az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PublicSubnet \
  --query "networkSecurityGroup.id" \
  --output tsv
```

### Windows PowerShell

```powershell
az network vnet subnet show `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PublicSubnet `
  --query "networkSecurityGroup.id" `
  --output tsv
```

---

# 12. Create PrivateSubnetNSG

## Linux / macOS

```bash
az network nsg create \
  --resource-group $RG \
  --name PrivateSubnetNSG \
  --location $LOCATION
```

## Windows PowerShell

```powershell
az network nsg create `
  --resource-group $RG `
  --name PrivateSubnetNSG `
  --location $LOCATION
```

---

# 13. Allow SSH TCP 22 on PrivateSubnetNSG

## Linux / macOS

```bash
az network nsg rule create \
  --resource-group $RG \
  --nsg-name PrivateSubnetNSG \
  --name Allow-SSH \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22
```

## Windows PowerShell

```powershell
az network nsg rule create `
  --resource-group $RG `
  --nsg-name PrivateSubnetNSG `
  --name Allow-SSH `
  --priority 100 `
  --direction Inbound `
  --access Allow `
  --protocol Tcp `
  --source-address-prefixes "*" `
  --destination-address-prefixes "*" `
  --destination-port-ranges 22
```

---

# 14. Allow HTTP TCP 80 on PrivateSubnetNSG

## Linux / macOS

```bash
az network nsg rule create \
  --resource-group $RG \
  --nsg-name PrivateSubnetNSG \
  --name Allow-HTTP \
  --priority 110 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 80
```

## Windows PowerShell

```powershell
az network nsg rule create `
  --resource-group $RG `
  --nsg-name PrivateSubnetNSG `
  --name Allow-HTTP `
  --priority 110 `
  --direction Inbound `
  --access Allow `
  --protocol Tcp `
  --source-address-prefixes "*" `
  --destination-address-prefixes "*" `
  --destination-port-ranges 80
```

The PrivateSubnetNSG now contains:

```text
Priority    Port    Action
--------    ----    ------
100         22      Allow
110         80      Allow
```

---

# 15. Attach PrivateSubnetNSG to PrivateSubnet

## Linux / macOS

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --network-security-group PrivateSubnetNSG
```

## Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --network-security-group PrivateSubnetNSG
```

Verify:

### Linux / macOS

```bash
az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --query "networkSecurityGroup.id" \
  --output tsv
```

### Windows PowerShell

```powershell
az network vnet subnet show `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --query "networkSecurityGroup.id" `
  --output tsv
```

---

# 16. Create Public Ubuntu VM

The VM will be created inside:

```text
VNet:       MyVNet
Subnet:     PublicSubnet
OS:         Ubuntu
Public IP:  Yes
```

## Linux / macOS

```bash
az vm create \
  --resource-group $RG \
  --name PublicVM \
  --image Ubuntu2204 \
  --vnet-name $VNET \
  --subnet PublicSubnet \
  --admin-username azureuser \
  --admin-username raman \
  --admin-password "Password@1234567"
  --public-ip-sku Standard
```

## Windows PowerShell

```powershell
az vm create `
  --resource-group $RG `
  --name PublicVM `
  --image Ubuntu2204 `
  --size Standard_D2s_v3 `
  --vnet-name $VNET `
   --subnet PublicSubnet `
   --admin-username raman `
   --admin-password "Password@1234567" `
   --public-ip-sku Standard```

Get the public IP:

### Linux / macOS

```bash
az vm show \
  --resource-group $RG \
  --name PublicVM \
  --show-details \
  --query publicIps \
  --output tsv
```

### Windows PowerShell

```powershell
az vm show `
  --resource-group $RG `
  --name PublicVM `
  --show-details `
  --query publicIps `
  --output tsv
```

Connect:

```bash
ssh azureuser@<PUBLIC-IP>
```

---

# 17. Create Private Ubuntu VM

The PrivateVM will be connected to `PrivateSubnet`.

It will **not have a public IP**.

## Linux / macOS

```bash
az vm create \
  --resource-group $RG \
  --name PrivateVM \
  --image Ubuntu2204 \
  --vnet-name $VNET \
  --subnet PrivateSubnet \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-address ""
```

## Windows PowerShell

```powershell
az vm create `
  --resource-group $RG `
  --name PrivateVM `
  --image Ubuntu2204 `
  --size Standard_D2s_v3 `
  --vnet-name $VNET `
   --subnet PrivateSubnet `
   --admin-username raman `
   --admin-password "Password@1234567" `
   --public-ip-sku Standard```
ress ""
```

Get the private IP:

### Linux / macOS

```bash
az vm show \
  --resource-group $RG \
  --name PrivateVM \
  --show-details \
  --query privateIps \
  --output tsv
```

### Windows PowerShell

```powershell
az vm show `
  --resource-group $RG `
  --name PrivateVM `
  --show-details `
  --query privateIps `
  --output tsv
```

Example:

```text
10.0.2.4
```

---

# 18. Find the VM NIC ID

Every VM has one or more NICs.

The relationship is:

```text
VM
 |
 +-- NIC
       |
       +-- IP Configuration
```

## PublicVM NIC ID

### Linux / macOS

```bash
az vm show \
  --resource-group $RG \
  --name PublicVM \
  --query "networkProfile.networkInterfaces[0].id" \
  --output tsv
```

### Windows PowerShell

```powershell
az vm show `
  --resource-group $RG `
  --name PublicVM `
  --query "networkProfile.networkInterfaces[0].id" `
  --output tsv
```

## PrivateVM NIC ID

### Linux / macOS

```bash
az vm show \
  --resource-group $RG \
  --name PrivateVM \
  --query "networkProfile.networkInterfaces[0].id" \
  --output tsv
```

### Windows PowerShell

```powershell
az vm show `
  --resource-group $RG `
  --name PrivateVM `
  --query "networkProfile.networkInterfaces[0].id" `
  --output tsv
```

---

# 19. List All NICs

This is often the easiest way to identify the NIC names.

## Linux / macOS

```bash
az network nic list \
  --resource-group $RG \
  --output table
```

## Windows PowerShell

```powershell
az network nic list `
  --resource-group $RG `
  --output table
```

You should see something similar to:

```text
Name             Location       ProvisioningState
---------------  -------------  -----------------
PublicVM-nic     centralindia   Succeeded
PrivateVM-nic    centralindia   Succeeded
```

---

# 20. Create NICNSG

Now create an NSG that will be attached directly to the NICs.

```text
NICNSG
 |
 +-- TCP 22 = DENY
```

## Linux / macOS

```bash
az network nsg create \
  --resource-group $RG \
  --name NICNSG \
  --location $LOCATION
```

## Windows PowerShell

```powershell
az network nsg create `
  --resource-group $RG `
  --name NICNSG `
  --location $LOCATION
```

---

# 21. Create Deny TCP 22 Rule

## Linux / macOS

```bash
az network nsg rule create \
  --resource-group $RG \
  --nsg-name NICNSG \
  --name Deny-SSH \
  --priority 100 \
  --direction Inbound \
  --access Deny \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22
```

## Windows PowerShell

```powershell
az network nsg rule create `
  --resource-group $RG `
  --nsg-name NICNSG `
  --name Deny-SSH `
  --priority 100 `
  --direction Inbound `
  --access Deny `
  --protocol Tcp `
  --source-address-prefixes "*" `
  --destination-address-prefixes "*" `
  --destination-port-ranges 22
```

Verify:

### Linux / macOS

```bash
az network nsg rule list \
  --resource-group $RG \
  --nsg-name NICNSG \
  --output table
```

### Windows PowerShell

```powershell
az network nsg rule list `
  --resource-group $RG `
  --nsg-name NICNSG `
  --output table
```

Expected:

```text
Priority    Name        Access    Direction    Port
--------    ----------  --------  -----------  ----
100         Deny-SSH    Deny      Inbound      22
```

---

# 22. Attach NICNSG to PublicVM NIC

The NIC name is assumed to be:

```text
PublicVM-nic
```

## Linux / macOS

```bash
az network nic update \
  --resource-group $RG \
  --name PublicVM-nic \
  --network-security-group NICNSG
```

## Windows PowerShell

```powershell
az network nic update `
  --resource-group $RG `
  --name PublicVM-nic `
  --network-security-group NICNSG
```

Verify:

### Linux / macOS

```bash
az network nic show \
  --resource-group $RG \
  --name PublicVM-nic \
  --query "networkSecurityGroup.id" \
  --output tsv
```

### Windows PowerShell

```powershell
az network nic show `
  --resource-group $RG `
  --name PublicVM-nic `
  --query "networkSecurityGroup.id" `
  --output tsv
```

---

# 23. Attach NICNSG to PrivateVM NIC

## Linux / macOS

```bash
az network nic update \
  --resource-group $RG \
  --name PrivateVM-nic \
  --network-security-group NICNSG
```

## Windows PowerShell

```powershell
az network nic update \
  --resource-group $RG \
  --name PrivateVM-nic \
  --network-security-group NICNSG
```

---

# 24. Understand NSG Processing

At this point, PublicVM has:

```text
Internet
   |
   v
PublicSubnet
   |
   v
PublicSubnetNSG
   |
   | TCP 22 = ALLOW
   |
   v
PublicVM NIC
   |
   v
NICNSG
   |
   | TCP 22 = DENY
   |
   v
PublicVM
```

The result is:

```text
PublicSubnetNSG
       |
       | TCP 22 → ALLOW
       |
       v
    NICNSG
       |
       | TCP 22 → DENY
       |
       v
     DENIED
```

The subnet NSG's **Allow** does not override the NIC NSG's **Deny**.

---

# 25. Important NSG Rule Concept

NSG rules are evaluated based on priority.

Lower number = higher priority.

Example:

```text
Priority 100 → evaluated before Priority 200
Priority 200 → evaluated before Priority 300
```

Example:

```text
Priority 100
Allow TCP 22

Priority 200
Deny TCP 22
```

Result:

```text
TCP 22 = ALLOW
```

because the priority 100 rule matches first.

In our lab:

```text
NICNSG

Priority 100
Deny TCP 22
```

Therefore SSH is denied by NICNSG.

---

# 26. Remove NSG from NIC

If you want to remove the NSG association from the NIC, use:

## Linux / macOS

```bash
az network nic update \
  --resource-group $RG \
  --name PublicVM-nic \
  --remove networkSecurityGroup
```

## Windows PowerShell

```powershell
az network nic update `
  --resource-group $RG `
  --name PublicVM-nic `
  --remove networkSecurityGroup
```

For PrivateVM:

### Linux / macOS

```bash
az network nic update \
  --resource-group $RG \
  --name PrivateVM-nic \
  --remove networkSecurityGroup
```

### Windows PowerShell

```powershell
az network nic update `
  --resource-group $RG `
  --name PrivateVM-nic `
  --remove networkSecurityGroup
```

Verify:

### Linux / macOS

```bash
az network nic show \
  --resource-group $RG \
  --name PublicVM-nic \
  --query networkSecurityGroup \
  --output json
```

### Windows PowerShell

```powershell
az network nic show `
  --resource-group $RG `
  --name PublicVM-nic `
  --query networkSecurityGroup `
  --output json
```

Expected:

```text
null
```

---

# 27. Remove NSG from Subnet

We can also remove the subnet-level NSG.

## Remove PublicSubnetNSG

### Linux / macOS

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PublicSubnet \
  --remove networkSecurityGroup
```

### Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PublicSubnet `
  --remove networkSecurityGroup
```

## Remove PrivateSubnetNSG

### Linux / macOS

```bash
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET \
  --name PrivateSubnet \
  --remove networkSecurityGroup
```

### Windows PowerShell

```powershell
az network vnet subnet update `
  --resource-group $RG `
  --vnet-name $VNET `
  --name PrivateSubnet `
  --remove networkSecurityGroup
```

---

# 28. Useful Verification Commands

## Show VNet

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

## List Subnets

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

## List NSGs

### Linux / macOS

```bash
az network nsg list \
  --resource-group $RG \
  --output table
```

### Windows PowerShell

```powershell
az network nsg list `
  --resource-group $RG `
  --output table
```

---

## List NSG Rules

### Linux / macOS

```bash
az network nsg rule list \
  --resource-group $RG \
  --nsg-name NICNSG \
  --output table
```

### Windows PowerShell

```powershell
az network nsg rule list `
  --resource-group $RG `
  --nsg-name NICNSG `
  --output table
```

---

## List VMs

### Linux / macOS

```bash
az vm list \
  --resource-group $RG \
  --output table
```

### Windows PowerShell

```powershell
az vm list `
  --resource-group $RG `
  --output table
```

---

## List NICs

### Linux / macOS

```bash
az network nic list \
  --resource-group $RG \
  --output table
```

### Windows PowerShell

```powershell
az network nic list `
  --resource-group $RG `
  --output table
```

---

# 29. Final Architecture

```text
                         Resource Group
                          MYRG-India
                               |
                               |
                            MyVNet
                         10.0.0.0/16
                               |
                +--------------+--------------+
                |                             |
                |                             |
         PublicSubnet                  PrivateSubnet
         10.0.10.0/24                 10.0.2.0/24
                |                             |
                |                             |
      PublicSubnetNSG                PrivateSubnetNSG
                |                             |
         Allow TCP 22                  Allow TCP 22
                                        Allow TCP 80
                |                             |
                |                             |
           PublicVM                     PrivateVM
                |                             |
         PublicVM-nic                  PrivateVM-nic
                |                             |
                +-------------+---------------+
                              |
                           NICNSG
                              |
                        Deny TCP 22
```

---

# 30. AZ-104 Exam Concepts

### VNet

Provides the virtual network.

```text
VNet = 10.0.0.0/16
```

### Subnet

Divides the VNet into smaller network segments.

```text
PublicSubnet  = 10.0.10.0/24
PrivateSubnet = 10.0.2.0/24
```

### NSG

Contains network security rules.

```text
NSG
 |
 +-- Allow
 |
 +-- Deny
```

### Subnet-Level NSG

```text
VNet
 |
 +-- Subnet
       |
       +-- NSG
```

### NIC-Level NSG

```text
VM
 |
 +-- NIC
       |
       +-- NSG
```

### Important

The VNet itself does **not** have an NSG directly attached.

The relationship is:

```text
VNet
 |
 +-- Subnet
       |
       +-- NSG
```

and:

```text
VM
 |
 +-- NIC
       |
       +-- NSG
```

---

# 31. Quick AZ-104 Memory Trick

```text
VNet
  ↓
Subnet
  ↓
NSG
```

and:

```text
VM
 ↓
NIC
 ↓
NSG
```

Remember:

```text
Subnet NSG = subnet-level security

NIC NSG = NIC-level security

NSG = security rules

NSG does not attach directly to a VNet
```

---

# 32. Linux vs Windows Azure CLI Syntax

The actual Azure CLI commands are identical.

Only line continuation changes:

```text
Linux/macOS
    \

Windows PowerShell
    `

Windows CMD
    ^
```

For example:

### Linux

```bash
az network nsg create \
  --resource-group MYRG-India \
  --name NICNSG \
  --location centralindia
```

### Windows PowerShell

```powershell
az network nsg create `
  --resource-group MYRG-India `
  --name NICNSG `
  --location centralindia
```

Both execute the same Azure CLI operation.

**You are still using Azure CLI in both cases.**
