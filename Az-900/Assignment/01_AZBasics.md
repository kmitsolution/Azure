### **Assignment: Create a Virtual Machine (VM), Virtual Machine Scale Set (VMSS), and Attach Disks with Load Balancer in Azure**

#### **Objective:**
In this assignment, you will demonstrate your ability to create a Virtual Machine (VM), configure a Virtual Machine Scale Set (VMSS), and set up a Load Balancer in Microsoft Azure. You will also attach additional disks to the virtual machines and configure the necessary settings for high availability.

#### **Prerequisites:**
1. Azure subscription or free trial account.
2. Basic understanding of Azure Portal and Azure CLI.
3. Familiarity with Virtual Machines, Virtual Machine Scale Sets (VMSS), Load Balancers, and Disks in Azure.

---

### **Tasks:**

#### **Part 1: Create a Virtual Machine (VM)**

1. **Log into the Azure Portal**:
   - Go to [Azure Portal](https://portal.azure.com/).

2. **Create a Resource Group** (if you don’t already have one):
   - Navigate to **Resource Groups** > **Add**.
   - Name the resource group (e.g., `RG-VM-LB`).
   - Choose a region (e.g., **East US**).

3. **Create a Virtual Machine**:
   - Go to **Virtual Machines** > **+ Add**.
   - Choose the following:
     - **Subscription**: Select your Azure subscription.
     - **Resource Group**: Select the resource group created earlier.
     - **Virtual Machine Name**: `VM-1`
     - **Region**: Same region as the resource group (e.g., **East US**).
     - **Image**: Select **Ubuntu 20.04 LTS** (or any other OS of your choice).
     - **Size**: Choose a VM size (e.g., **B1s** for testing).
     - **Authentication Type**: SSH public key (create a key pair or use an existing one).
   - Under **Disks**, choose **Standard SSD** for your OS disk.

4. **Configure Networking**:
   - Create a new **Virtual Network** (VNet) or select an existing one.
   - Set the **Subnet** to an existing one or create a new one.
   - Leave other settings default for now.

5. **Create the Virtual Machine**:
   - Review your settings and click **Create**.

---

#### **Part 2: Create a Virtual Machine Scale Set (VMSS)**

1. **Navigate to VMSS**:
   - In the Azure Portal, search for **Virtual Machine Scale Sets**.

2. **Create a Virtual Machine Scale Set**:
   - Click on **+ Add**.
   - Enter the following details:
     - **Subscription**: Select your Azure subscription.
     - **Resource Group**: Use the same resource group as before (e.g., `RG-VM-LB`).
     - **Region**: Choose the same region (e.g., **East US**).
     - **Name**: `VMSS-1`
     - **VM Size**: Choose the same size as your VM (e.g., **B1s**).
     - **Image**: Select **Ubuntu 20.04 LTS** (or your preferred OS).
     - **Instance count**: Choose `2` or more (for high availability).

3. **Configure Scaling and Load Balancer**:
   - Under **Load Balancer**, select **Create a new Load Balancer**.
     - This will automatically configure the backend pool and health probe for your scale set.
   - Choose **Availability Zones** if you need additional redundancy.
   - Click **Next** to configure networking.
   
4. **Configure Networking**:
   - Create or select an existing **Virtual Network** and **Subnet**.
   - Ensure that the Load Balancer is configured to route traffic to your VMSS instances.

5. **Create the Scale Set**:
   - Review your settings and click **Create**.

---

#### **Part 3: Attach Additional Disks to VM and VMSS Instances**

1. **Attach Additional Disk to VM**:
   - Navigate to **Virtual Machines** > `VM-1` > **Disks**.
   - Click **+ Add Data Disk**.
   - Choose the disk type (e.g., **Standard SSD**) and size (e.g., 10 GB).
   - Click **Save** to attach the disk.

2. **Attach Additional Disks to VMSS Instances**:
   - Navigate to **Virtual Machine Scale Sets** > `VMSS-1` > **Instances**.
   - Select one of the instances.
   - Click **+ Add Data Disk** to attach a new data disk (e.g., 10 GB).
   - Repeat for other instances in the scale set.
   
   > **Note:** If using the **Azure CLI** or **ARM Templates**, you can automate the process of attaching disks to scale set instances.

---

#### **Part 4: Test the Load Balancer**

1. **Access the Load Balancer**:
   - Go to **Load Balancers** in the Azure Portal.
   - Select the Load Balancer you created with your VMSS.

2. **Check Backend Pool**:
   - Ensure that the VMSS instances are listed in the **Backend Pool**.
   - Verify that the Load Balancer health probe is successful for each instance.

3. **Test the Traffic Flow**:
   - Note the **public IP address** assigned to the Load Balancer.
   - Open a browser or use a tool like **curl** to send traffic to the Load Balancer’s IP address and verify that it is evenly distributed between the VMSS instances.

---

### **Part 5: Clean Up Resources**

1. **Delete Resources**:
   - Once testing is complete, delete all the resources you created (VM, VMSS, Load Balancer, Disks, Resource Group) to avoid unnecessary costs.

   **To delete:**
   - Navigate to **Resource Groups** in the Azure Portal.
   - Select the resource group (`RG-VM-LB`).
   - Click **Delete Resource Group** and confirm.

---
### **Evaluation Criteria:**

1. **Correctness of VM and VMSS Configuration**: The VM should be correctly created and configured, and the VMSS should be set up with at least two instances.
2. **Proper Disk Attachment**: Additional disks should be successfully attached to both the VM and VMSS instances.
3. **Load Balancer Functionality**: The Load Balancer should be correctly distributing traffic across the VMSS instances.
4. **Clean-Up**: Proper clean-up of resources after the assignment is completed.

---

### **Resources:**
- [Create a Virtual Machine in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/quick-create-portal)
- [Create a Virtual Machine Scale Set in Azure](https://learn.microsoft.com/en-us/azure/virtual-machine-scale-sets/quick-create-portal)
- [Azure Load Balancer Overview](https://learn.microsoft.com/en-us/azure/load-balancer/load-balancer-overview)
- [Attach and Manage Disks on Azure VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/disks)

---

This assignment is designed to give you hands-on experience with creating key Azure resources and configuring high-availability and scalability features. It also emphasizes the importance of managing storage and load balancing for production workloads in Azure.
