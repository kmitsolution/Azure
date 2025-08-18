# Azure Disk Encryption
Azure Disk Encryption is a service that helps protect your data by encrypting the disks used by Azure Virtual Machines. Here's a detailed overview of the Azure Disk Encryption process:

## Azure Disk Encryption Overview

<b>Purpose:</b> To secure data at rest on Azure Virtual Machine (VM) disks by using BitLocker (for Windows VMs) or DM-Crypt (for Linux VMs).

### Key Components
#### Azure Key Vault:

A secure store for your encryption keys. You need to create a Key Vault and configure access policies.

#### Encryption Key:

Managed by Azure Key Vault. This key is used to encrypt the VM disks.

## Step-by-step guide for encrypting an Azure Disk using Azure Disk Encryption:

### Azure Disk Encryption Process

1. **Create a Virtual Machine (VM)**:
   - Start by creating a new VM in Azure or select an existing VM where you want to attach the disk.

2. **Attach a Disk to the VM**:
   - Go to the VM you created or selected.
   - Navigate to the "Disks" section.
   - Click on "Add data disk" to attach a new disk or select an existing disk to attach.

3. **Select the Disk for Encryption**:
   - Once the disk is attached, navigate to the "Disks" section again.
   - Click on the disk you want to encrypt.

4. **Access Additional Settings**:
   - In the disk settings page, go to the "Additional Settings" tab.

5. **Configure Encryption**:
   - Choose the disks you want to encrypt (Data disk, OS disk, or both).
   - Select the Key Vault and the encryption key that you want to use. This key vault must be configured in advance and accessible.

6. **Complete Encryption Setup**:
   - Confirm and apply the settings. The disk will start the encryption process.

7. **Verify Encryption**:
   - Check the status of the disk to ensure that the encryption process has completed successfully.

### Additional Considerations

- **Key Vault Access**: Ensure that the VM has the necessary permissions to access the Key Vault and the specified encryption keys.
- **Compliance and Security**: Regularly rotate your keys and monitor the encryption status for compliance and security best practices.

Feel free to ask if you have any specific questions or need further details!
