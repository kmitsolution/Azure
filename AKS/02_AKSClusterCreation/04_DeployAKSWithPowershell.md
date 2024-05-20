# AKS Installation using Powershell commands

To create an Azure Kubernetes Service (AKS) cluster using PowerShell commands, you can utilize the Azure PowerShell module. Below are the steps to create an AKS cluster using PowerShell:

1. **Install and Import Azure PowerShell Module**: If you haven't already installed the Azure PowerShell module, you can install it using the following command:

   ```powershell
   Install-Module -Name Az -AllowClobber -Force
   ```

   After installing the module, import it using:

   ```powershell
   Import-Module Az
   ```

2. **Connect to Azure**: Use `Connect-AzAccount` to authenticate with your Azure account:

   ```powershell
   Connect-AzAccount
   ```

   Follow the prompts to sign in with your Azure credentials.

3. **Set the Subscription**: If you have multiple subscriptions, set the subscription you want to use:

   ```powershell
   Set-AzContext -SubscriptionId "<subscription-id>"
   ```

   Replace `<subscription-id>` with the ID of your Azure subscription.

4. **Create Resource Group**: Create a resource group where your AKS cluster will be deployed:

   ```powershell
   New-AzResourceGroup -Name "myResourceGroup" -Location "East US"
   ```

   Replace `"myResourceGroup"` and `"East US"` with your desired resource group name and Azure region.

5. **Create AKS Cluster**: Use `New-AzAks` to create the AKS cluster:

   ```powershell
   New-AzAks -ResourceGroupName "myResourceGroup" -Name "myAKSCluster" -NodeCount 3 -NodeVmSize "Standard_DS2_v2" -Location "East US"
   ```

   Adjust the parameters (`-Name`, `-NodeCount`, `-NodeVmSize`, etc.) according to your requirements.

6. **Get Credentials**: After the cluster is created, retrieve the credentials to connect to the cluster:

   ```powershell
   Get-AzAksCredential -ResourceGroupName "myResourceGroup" -Name "myAKSCluster" -Admin
   ```

   This command will add the AKS cluster's credentials to your Kubernetes configuration (`~/.kube/config`).

Now you've successfully created an AKS cluster using PowerShell commands. You can interact with the cluster using `kubectl` or deploy workloads to it.
