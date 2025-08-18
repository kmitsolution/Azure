
### Create Linux VM Using Azure Portal:

1. **Sign in to Azure Portal**:
   - Navigate to [Azure Portal](https://portal.azure.com) and sign in with your Azure account.

2. **Click "Create a Resource"**:
   - Click on the "+ Create a resource" button in the upper-left corner.

3. **Search for "Ubuntu Server"**:
   - In the search bar, type "Ubuntu Server".
   - Select the "Ubuntu Server" option from the search results.

4. **Configure VM**:
   - Fill out the required information such as Resource Group, VM Name, Username, and Password.
   - Choose your preferred Subscription, Resource Group, Region, and VM Size.
   - Configure additional settings such as Disk, Networking, Management, etc. as per your requirements.

5. **Review and Create**:
   - Review the summary and click "Review + Create".
   - After validation passes, click "Create" to deploy the VM.

### Create Linux VM Using Azure CLI:

Here's how you can create a Linux VM using Azure CLI:

```bash
az vm create \
    --resource-group YourResourceGroup \
    --name YourVMName \
    --image UbuntuLTS \
    --admin-username YourAdminUsername \
    --admin-password YourAdminPassword \
    --size Standard_DS1_v2 \
    --location eastus
    --zone 1
```

Replace placeholders (`YourResourceGroup`, `YourVMName`, `YourAdminUsername`, `YourAdminPassword`) with your actual values.

### Create Linux VM Using PowerShell:

Here's how you can create a Linux VM using PowerShell:

```powershell
New-AzVm `
    -ResourceGroupName YourResourceGroup `
    -Name YourVMName `
    -Image "Canonical:UbuntuServer:18.04-LTS-gen2:latest" `
    -Credential (Get-Credential) `
    -Size Standard_DS1_v2 `
    -Location "East US" `
    -OpenPorts 22
```

Replace placeholders (`YourResourceGroup`, `YourVMName`) with your actual values.

These commands will create a Linux VM in Azure with the specified configurations using Azure CLI and PowerShell, respectively. Adjust the parameters as needed for your environment.
