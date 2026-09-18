# AKS Cluster Installation Using Azure CLI

## 1. Objective

In this lesson, we will create an **Azure Kubernetes Service (AKS)** cluster using Azure CLI.

We will:

1. Define reusable variables.
2. Create an Azure Resource Group.
3. Create an AKS cluster.
4. Create a node pool with **2 worker nodes**.
5. Use `Standard_D2s_v5` as the VM size.
6. Configure `kubectl`.
7. Verify the Kubernetes nodes.
8. Verify the node pool.
9. Identify the AKS infrastructure resource group.
10. Examine the Azure resources created for the AKS cluster.

The commands are provided for both:

* **PowerShell**
* **Bash**

---

# 2. AKS Architecture

The AKS cluster can be visualized as:

```text
                         Azure
                           |
                           v
                    +-------------+
                    | AKS Cluster |
                    +-------------+
                           |
             +-------------+-------------+
             |                           |
             v                           v
       Control Plane                Node Pool
       Azure Managed              Customer Managed
             |                           |
       +-----+------+              +-----+-----+
       |     |      |              |           |
      API  Scheduler Controller   Node 1     Node 2
     Server         Manager         |           |
                                    |           |
                                   Pods        Pods
                                    |
                               Containers
```

### Control Plane

The control plane is the **brain of Kubernetes**.

It contains components such as:

* Kubernetes API Server
* Scheduler
* Controller Manager
* etcd

In AKS, the control plane is **managed by Azure**.

### Node Pool

The node pool contains the Kubernetes worker nodes.

For this lab:

```text
Node Pool: agentpool

        +----------------+
        | Node 1         |
        | D2s_v5         |
        +----------------+

        +----------------+
        | Node 2         |
        | D2s_v5         |
        +----------------+
```

Application Pods run on these worker nodes.

---

# 3. Prerequisites

Before starting, make sure you have:

* An Azure subscription
* Azure CLI installed
* Access to the Azure subscription
* `kubectl` available

Check Azure CLI:

### PowerShell / Bash

```bash
az version
```

Check `kubectl`:

```bash
kubectl version --client
```

---

# 4. Login to Azure

## PowerShell

```powershell
az login
```

## Bash

```bash
az login
```

The command is the same in both environments.

After login, verify the subscription:

```bash
az account show --output table
```

---

# 5. Select Azure Subscription

If you have multiple Azure subscriptions, first list them.

### PowerShell / Bash

```bash
az account list --output table
```

Set the required subscription.

## PowerShell

```powershell
$SUBSCRIPTION_ID = "<YOUR-SUBSCRIPTION-ID>"

az account set --subscription $SUBSCRIPTION_ID
```

## Bash

```bash
SUBSCRIPTION_ID="<YOUR-SUBSCRIPTION-ID>"

az account set --subscription "$SUBSCRIPTION_ID"
```

Verify:

```bash
az account show --output table
```

---

# 6. Define Variables

This is an important practice.

Instead of hardcoding values directly into every Azure CLI command, we define variables.

For example:

```text
Location
Resource Group
AKS Cluster Name
Node Pool Name
Node Count
VM Size
```

---

## PowerShell Variables

```powershell
$LOCATION = "centralindia"
$RG = "AKS-RG"
$AKS_NAME = "myakscluster"

$NODE_POOL = "agentpool"
$NODE_COUNT = 2
$NODE_SIZE = "Standard_D2s_v5"
```

---

## Bash Variables

```bash
LOCATION="centralindia"
RG="AKS-RG"
AKS_NAME="myakscluster"

NODE_POOL="agentpool"
NODE_COUNT=2
NODE_SIZE="Standard_D2s_v5"
```

Notice the difference.

### PowerShell

```powershell
$LOCATION = "centralindia"
```

### Bash

```bash
LOCATION="centralindia"
```

---

# 7. Why Use Variables?

Without variables, we might write:

```bash
az aks create \
  --resource-group AKS-RG \
  --name myakscluster \
  --location centralindia
```

This works, but it is not very reusable.

With variables:

```bash
az aks create \
  --resource-group "$RG" \
  --name "$AKS_NAME" \
  --location "$LOCATION"
```

we can change the environment simply by changing the variables.

For example:

```text
LOCATION = eastus
RG       = AKS-Training-RG
AKS_NAME = trainingaks
```

without changing the actual commands.

---

# 8. Create Resource Group

The first Azure resource we create is the Resource Group.

---

## PowerShell

```powershell
az group create `
  --name $RG `
  --location $LOCATION
```

PowerShell uses the **backtick (`)** for multiline commands.

---

## Bash

```bash
az group create \
  --name "$RG" \
  --location "$LOCATION"
```

Bash uses the **backslash (`\`)** for multiline commands.

---

## Verify Resource Group

PowerShell and Bash:

```bash
az group show \
  --name "$RG" \
  --output table
```

PowerShell can also use:

```powershell
az group show `
  --name $RG `
  --output table
```

Expected result:

```text
Location      Name
------------  -------
centralindia  AKS-RG
```

---

# 9. Check VM Size Availability

For this lab we are using:

```text
Standard_D2s_v5
```

This is a reasonable general-purpose size for an AKS training cluster.

However, **no VM size can be guaranteed in every Azure region/subscription**, because quota and regional capacity can vary.

We can check whether the size is available in the selected region.

---

## PowerShell

```powershell
az vm list-sizes `
  --location $LOCATION `
  --query "[?name=='$NODE_SIZE']" `
  --output table
```

---

## Bash

```bash
az vm list-sizes \
  --location "$LOCATION" \
  --query "[?name=='$NODE_SIZE']" \
  --output table
```

If the size is returned, it exists in that region.

You should still be aware of **vCPU quota and regional capacity** when creating the actual cluster.

---

# 10. Create AKS Cluster

Now we create the AKS cluster.

The important parameters are:

```text
--resource-group
--name
--location
--node-count
--node-vm-size
--nodepool-name
--generate-ssh-keys
```

---

## PowerShell

```powershell
az aks create `
  --resource-group $RG `
  --name $AKS_NAME `
  --location $LOCATION `
  --node-count $NODE_COUNT `
  --node-vm-size $NODE_SIZE `
  --nodepool-name $NODE_POOL `
  --generate-ssh-keys
```

---

## Bash

```bash
az aks create \
  --resource-group "$RG" \
  --name "$AKS_NAME" \
  --location "$LOCATION" \
  --node-count "$NODE_COUNT" \
  --node-vm-size "$NODE_SIZE" \
  --nodepool-name "$NODE_POOL" \
  --generate-ssh-keys
```

---

# 11. Understanding the AKS Create Command

Let's understand each parameter.

### Resource Group

```text
--resource-group $RG
```

Specifies the Resource Group where the AKS resource will be created.

---

### AKS Name

```text
--name $AKS_NAME
```

Specifies the name of the AKS cluster.

Example:

```text
myakscluster
```

---

### Location

```text
--location $LOCATION
```

Specifies the Azure region.

Example:

```text
centralindia
```

---

### Node Count

```text
--node-count $NODE_COUNT
```

We defined:

```text
NODE_COUNT = 2
```

Therefore:

```text
Node Pool
   |
   +-- Node 1
   |
   +-- Node 2
```

---

### Node VM Size

```text
--node-vm-size $NODE_SIZE
```

We defined:

```text
NODE_SIZE = Standard_D2s_v5
```

Therefore each worker node uses that VM size.

---

### Node Pool Name

```text
--nodepool-name $NODE_POOL
```

We defined:

```text
NODE_POOL = agentpool
```

Therefore:

```text
AKS
 |
 +-- agentpool
       |
       +-- Node 1
       +-- Node 2
```

---

### Generate SSH Keys

```text
--generate-ssh-keys
```

Azure CLI generates SSH keys if required for the AKS node configuration.

---

# 12. Get AKS Credentials

After the cluster has been created, we need to configure `kubectl`.

## PowerShell

```powershell
az aks get-credentials `
  --resource-group $RG `
  --name $AKS_NAME
```

## Bash

```bash
az aks get-credentials \
  --resource-group "$RG" \
  --name "$AKS_NAME"
```

This updates your local Kubernetes configuration, commonly called the **kubeconfig**.

Now `kubectl` knows how to communicate with your AKS cluster.

---

# 13. Verify Kubernetes Nodes

Run:

```bash
kubectl get nodes
```

Expected output will look similar to:

```text
NAME                                STATUS   ROLES
aks-agentpool-xxxxx-vmss000000      Ready    <none>
aks-agentpool-xxxxx-vmss000001      Ready    <none>
```

We can also get more information:

```bash
kubectl get nodes -o wide
```

You should see **two worker nodes** in the `Ready` state.

---

# 14. Verify Node Pool

Use Azure CLI:

## PowerShell

```powershell
az aks nodepool list `
  --resource-group $RG `
  --cluster-name $AKS_NAME `
  --output table
```

## Bash

```bash
az aks nodepool list \
  --resource-group "$RG" \
  --cluster-name "$AKS_NAME" \
  --output table
```

You should see something similar to:

```text
Name        Count    VMSize
----------  -------  ----------------
agentpool   2        Standard_D2s_v5
```

---

# 15. Understand the Architecture Now

At this point, our environment looks like:

```text
                    Azure Subscription
                           |
                           v
                       AKS-RG
                           |
                           v
                    myakscluster
                           |
              +------------+------------+
              |                         |
              v                         v
       Control Plane               agentpool
       Azure Managed                   |
                                  +----+----+
                                  |         |
                                  v         v
                               Node 1    Node 2
                                  |         |
                                 Pods      Pods
```

The control plane is managed by Azure.

The two worker nodes belong to the `agentpool` node pool.

---

# 16. Find the AKS Infrastructure Resource Group

When AKS is created, Azure creates a separate **managed infrastructure resource group** for the cluster.

Its name typically looks like:

```text
MC_<ResourceGroup>_<AKSCluster>_<Region>
```

For example:

```text
MC_AKS-RG_myakscluster_centralindia
```

Rather than assuming the name, retrieve it dynamically.

---

## PowerShell

```powershell
$NODE_RG = az aks show `
  --resource-group $RG `
  --name $AKS_NAME `
  --query nodeResourceGroup `
  --output tsv

Write-Host "AKS Infrastructure Resource Group: $NODE_RG"
```

---

## Bash

```bash
NODE_RG=$(az aks show \
  --resource-group "$RG" \
  --name "$AKS_NAME" \
  --query nodeResourceGroup \
  --output tsv)

echo "AKS Infrastructure Resource Group: $NODE_RG"
```

This is a good example of using the output of one Azure CLI command as a variable for another command.

---

# 17. List Infrastructure Resources

Now list the resources in the managed infrastructure resource group.

## PowerShell

```powershell
az resource list `
  --resource-group $NODE_RG `
  --output table
```

## Bash

```bash
az resource list \
  --resource-group "$NODE_RG" \
  --output table
```

Depending on the AKS configuration, you may see resources such as:

```text
Virtual Machine Scale Set
Managed Disk
Network Interface
Load Balancer
Public IP
```

---

# 18. Understanding the Infrastructure Resource Group

The architecture now looks like:

```text
                   Your Resource Group
                         AKS-RG
                           |
                           v
                    AKS Resource
                    myakscluster
                           |
                           |
                           v
              Managed Infrastructure RG
              MC_AKS-RG_myakscluster_...
                           |
              +------------+------------+
              |            |            |
              v            v            v
             VMSS        Disks      Networking
              |                         |
              |                    +----+----+
              |                    |         |
              v                    v         v
            Nodes               NIC       Load Balancer
              |
              v
             Pods
```

The managed infrastructure resource group is primarily an implementation detail of AKS and should not be treated as a normal application resource group.

---

# 19. Complete PowerShell Script

You can use the following as your complete AKS lab script.

```powershell
# ==========================================================
# AKS CLUSTER CREATION - POWERSHELL
# ==========================================================

# ----------------------------------------------------------
# Variables
# ----------------------------------------------------------

$LOCATION = "centralindia"
$RG = "AKS-RG"
$AKS_NAME = "myakscluster"

$NODE_POOL = "agentpool"
$NODE_COUNT = 2
$NODE_SIZE = "Standard_D2s_v5"


# ----------------------------------------------------------
# 1. Login
# ----------------------------------------------------------

az login


# ----------------------------------------------------------
# 2. Select Subscription - Optional
# ----------------------------------------------------------

# $SUBSCRIPTION_ID = "<YOUR-SUBSCRIPTION-ID>"
# az account set --subscription $SUBSCRIPTION_ID


# ----------------------------------------------------------
# 3. Create Resource Group
# ----------------------------------------------------------

az group create `
  --name $RG `
  --location $LOCATION


# ----------------------------------------------------------
# 4. Create AKS Cluster
# ----------------------------------------------------------

az aks create `
  --resource-group $RG `
  --name $AKS_NAME `
  --location $LOCATION `
  --node-count $NODE_COUNT `
  --node-vm-size $NODE_SIZE `
  --nodepool-name $NODE_POOL `
  --generate-ssh-keys


# ----------------------------------------------------------
# 5. Get AKS Credentials
# ----------------------------------------------------------

az aks get-credentials `
  --resource-group $RG `
  --name $AKS_NAME


# ----------------------------------------------------------
# 6. Verify Nodes
# ----------------------------------------------------------

Write-Host "======================================="
Write-Host "AKS Nodes"
Write-Host "======================================="

kubectl get nodes -o wide


# ----------------------------------------------------------
# 7. Verify Node Pool
# ----------------------------------------------------------

Write-Host "======================================="
Write-Host "AKS Node Pools"
Write-Host "======================================="

az aks nodepool list `
  --resource-group $RG `
  --cluster-name $AKS_NAME `
  --output table


# ----------------------------------------------------------
# 8. Get Infrastructure Resource Group
# ----------------------------------------------------------

$NODE_RG = az aks show `
  --resource-group $RG `
  --name $AKS_NAME `
  --query nodeResourceGroup `
  --output tsv

Write-Host "======================================="
Write-Host "Infrastructure Resource Group"
Write-Host "======================================="

Write-Host $NODE_RG


# ----------------------------------------------------------
# 9. List Infrastructure Resources
# ----------------------------------------------------------

Write-Host "======================================="
Write-Host "Infrastructure Resources"
Write-Host "======================================="

az resource list `
  --resource-group $NODE_RG `
  --output table
```

---

# 20. Complete Bash Script

```bash
#!/bin/bash

# ==========================================================
# AKS CLUSTER CREATION - BASH
# ==========================================================

# ----------------------------------------------------------
# Variables
# ----------------------------------------------------------

LOCATION="centralindia"
RG="AKS-RG"
AKS_NAME="myakscluster"

NODE_POOL="agentpool"
NODE_COUNT=2
NODE_SIZE="Standard_D2s_v5"


# ----------------------------------------------------------
# 1. Login
# ----------------------------------------------------------

az login


# ----------------------------------------------------------
# 2. Select Subscription - Optional
# ----------------------------------------------------------

# SUBSCRIPTION_ID="<YOUR-SUBSCRIPTION-ID>"
# az account set --subscription "$SUBSCRIPTION_ID"


# ----------------------------------------------------------
# 3. Create Resource Group
# ----------------------------------------------------------

az group create \
  --name "$RG" \
  --location "$LOCATION"


# ----------------------------------------------------------
# 4. Create AKS Cluster
# ----------------------------------------------------------

az aks create \
  --resource-group "$RG" \
  --name "$AKS_NAME" \
  --location "$LOCATION" \
  --node-count "$NODE_COUNT" \
  --node-vm-size "$NODE_SIZE" \
  --nodepool-name "$NODE_POOL" \
  --generate-ssh-keys


# ----------------------------------------------------------
# 5. Get AKS Credentials
# ----------------------------------------------------------

az aks get-credentials \
  --resource-group "$RG" \
  --name "$AKS_NAME"


# ----------------------------------------------------------
# 6. Verify Nodes
# ----------------------------------------------------------

echo "======================================="
echo "AKS Nodes"
echo "======================================="

kubectl get nodes -o wide


# ----------------------------------------------------------
# 7. Verify Node Pool
# ----------------------------------------------------------

echo "======================================="
echo "AKS Node Pools"
echo "======================================="

az aks nodepool list \
  --resource-group "$RG" \
  --cluster-name "$AKS_NAME" \
  --output table


# ----------------------------------------------------------
# 8. Get Infrastructure Resource Group
# ----------------------------------------------------------

NODE_RG=$(az aks show \
  --resource-group "$RG" \
  --name "$AKS_NAME" \
  --query nodeResourceGroup \
  --output tsv)

echo "======================================="
echo "Infrastructure Resource Group"
echo "======================================="

echo "$NODE_RG"


# ----------------------------------------------------------
# 9. List Infrastructure Resources
# ----------------------------------------------------------

echo "======================================="
echo "Infrastructure Resources"
echo "======================================="

az resource list \
  --resource-group "$NODE_RG" \
  --output table
```

---

# 21. PowerShell vs Bash — Quick Reference

| Operation                  | PowerShell            | Bash                |
| -------------------------- | --------------------- | ------------------- |
| Variable                   | `$RG = "AKS-RG"`      | `RG="AKS-RG"`       |
| Use variable               | `$RG`                 | `"$RG"`             |
| Multiline                  | `` ` ``               | `\`                 |
| Command output to variable | `$NODE_RG = az ...`   | `NODE_RG=$(az ...)` |
| Print variable             | `Write-Host $NODE_RG` | `echo "$NODE_RG"`   |
| Comments                   | `#`                   | `#`                 |

The **Azure CLI commands themselves are essentially the same**. The primary differences are the shell's variable and multiline-command syntax.

---

# 22. Final Environment

After completing this lab, your environment should conceptually look like:

```text
                    Azure Subscription
                           |
                           |
                    +------+------+
                    |             |
                    v             v
                 AKS-RG       Other Resources
                    |
                    v
              AKS Cluster
             myakscluster
                    |
        +-----------+-----------+
        |                       |
        v                       v
  Azure Managed             agentpool
  Control Plane                 |
                         +-------+-------+
                         |               |
                         v               v
                      Node 1          Node 2
                         |               |
                       Pods            Pods
                         |
                    Containers


             Managed Infrastructure RG
                        |
              +---------+---------+
              |         |         |
             VMSS      Disks    Networking
              |                   |
            Nodes            NIC / LB / IP
```

### The key takeaway

> **AKS = Azure-managed Kubernetes control plane + customer-managed worker node pools and workloads.**

For the next practical lesson, a natural progression is **deploy an NGINX application to these two nodes, expose it using a Kubernetes `Service` of type `LoadBalancer`, and then map the Kubernetes Service to the Azure Load Balancer/Public IP that appears in the infrastructure resource group.**
