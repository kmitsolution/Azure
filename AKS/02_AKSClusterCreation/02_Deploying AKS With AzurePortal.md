Here's a step-by-step guide on how to create an Azure Kubernetes Service (AKS) cluster using the Azure portal:

1. **Sign in to the Azure Portal**:
   - Go to https://portal.azure.com and sign in with your Azure account credentials.

2. **Navigate to AKS**:
   - In the Azure portal, click on the "Create a resource" button (the plus sign "+") in the upper-left corner.
   - In the search bar, type "Kubernetes Service" and select "Kubernetes Service" from the search results.
   - <img width="917" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/8e72ae4e-b3c7-445d-8f14-7e16d3009a32">


3. **Create AKS Cluster**:
   - In the "Kubernetes Service" blade, click on the "Create" button.
     <img width="903" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/090f2e07-3640-4bab-8166-23c4f041c538">

4. **Basics**:
   - In the "Basics" tab, fill in the required information:
     - Subscription: Choose your Azure subscription.
     - Resource group: Select an existing resource group or create a new one.
     - Kubernetes cluster name: Enter a unique name for your AKS cluster.
     - Region: Choose the Azure region where you want to deploy the cluster.
     - Kubernetes version: Select the desired Kubernetes version.

5. **Node Pools**:
   - In the "Node Pools" tab, configure the node pool settings:
     - Virtual Machine Size: Choose the VM size for the cluster nodes.
     - Node count: Specify the initial number of nodes in the node pool.
     - Node pool name: Enter a name for the node pool.
     - Node count: Specify the number of nodes in the node pool.
     - Availability Zones: Choose whether to enable Availability Zones for node distribution.

6. **Authentication** (Optional):
   - In the "Authentication" tab, configure the authentication method for the AKS cluster:
     - Enable Azure Active Directory integration (optional).
     - Choose whether to enable Role-Based Access Control (RBAC).

7. **Networking**:
   - In the "Networking" tab, configure the network settings for the AKS cluster:
     - Virtual Network: Select the existing VNet or create a new one.
     - Subnet: Select the subnet within the VNet to deploy the AKS cluster.

8. **Monitoring** (Optional):
   - In the "Monitoring" tab, configure monitoring settings for the AKS cluster:
     - Enable Azure Monitor container monitoring.
     - Enable Log Analytics integration.

9. **Review + Create**:
   - Review the configuration settings for the AKS cluster.
   - Click on the "Review + Create" button to validate the configuration.
   - After validation passes, click on the "Create" button to provision the AKS cluster.

10. **Deployment**:
    - Wait for the AKS cluster deployment to complete. This process may take several minutes.

11. **Access AKS Cluster**:
    - Once the deployment is successful, navigate to the AKS cluster overview page in the Azure portal to access details about the cluster.
    - You can also use the Azure CLI or Azure Cloud Shell to interact with the AKS cluster.

That's it! You've successfully created an Azure Kubernetes Service (AKS) cluster using the Azure portal. You can now deploy and manage containerized applications on the AKS cluster.
