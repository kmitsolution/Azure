# Server-side encryption
Server-side encryption with platform-managed and customer-managed keys, and then I'll refine the setup steps for customer-managed keys in Azure.

### **Platform-Managed Keys vs. Customer-Managed Keys**

1. **Platform-Managed Keys (PMK)**:
   - **Definition**: These are encryption keys managed by the cloud service provider (e.g., Azure). The provider is responsible for key lifecycle management, including key generation, rotation, and retirement.
   - **Use Case**: Suitable for users who prefer not to manage their own encryption keys and want to rely on the cloud provider's security infrastructure and practices.

2. **Customer-Managed Keys (CMK)**:
   - **Definition**: These are encryption keys that customers create and manage themselves. They have more control over key management, including key creation, rotation, and access permissions.
   - **Use Case**: Preferred by users who need additional control over their encryption keys for compliance, security, or regulatory reasons.

### **Steps to Set Up Customer-Managed Key in Azure**

1. **Create a Key Vault in Azure**:
   - **Description**: Azure Key Vault is a service that safeguards cryptographic keys and secrets used by cloud applications and services.
   - **How-To**:
     - Go to the Azure portal.
     - Navigate to "Create a resource" and search for "Key Vault".
     - Click "Create" and provide necessary details (e.g., resource group, Key Vault name, region).

2. **Generate/Import a Key with Specified Algorithm**:
   - **Description**: You can either generate a new key or import an existing one into Azure Key Vault.
   - **How-To**:
     - In the Azure portal, go to your Key Vault.
     - Navigate to "Keys" under the "Settings" section.
     - Click "Generate" to create a new key or "Import" to upload an existing key.
     - Specify the key type (e.g., RSA, EC) and key size/algorithm details.

3. **Create a Virtual Machine**:
   - **Description**: Deploy a virtual machine that will use the customer-managed key for encryption.
   - **How-To**:
     - Go to the Azure portal.
     - Navigate to "Virtual Machines" and click "Create".
     - Fill in the required details such as VM size, image, and network settings.

4. **Create a Disk Encryption Set and Specify the Key Vault Service**:
   - **Description**: A Disk Encryption Set (DES) uses a specified Key Vault key to encrypt the virtual machine's disks.
   - **How-To**:
     - In the Azure portal, navigate to "Create a resource" and search for "Disk Encryption Set".
     - Click "Create" and enter the required details.
     - In the "Key Vault" field, select the Key Vault you created earlier.
     - Select the key from the Key Vault to be used for encryption.

5. **Select the OS Disk and Configure Encryption with Customer-Managed Key**:
   - **Description**: Apply the Disk Encryption Set to the OS disk of your virtual machine.
   - **How-To**:
     - Go to your Virtual Machine's page in the Azure portal.
     - Under "Settings", find "Disks".
     - Select the OS disk and click on "Encryption" settings.
     - Choose "Customer-managed key" and select the Disk Encryption Set you created earlier.
     - Save the changes to apply encryption.

By following these steps, you can effectively set up and manage server-side encryption using customer-managed keys in Azure. This ensures that your data is encrypted with keys you control, providing an additional layer of security and compliance.
