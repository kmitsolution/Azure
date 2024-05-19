# AKS Installation using Azure CLI
To install and uninstall an Azure Kubernetes Service (AKS) cluster using Azure CLI, you can follow these steps:

### Installation:

1. **Log in to Azure**: Use the `az login` command to authenticate with your Azure account.

   ```bash
   az login
   ```

2. **Set Subscription (Optional)**: If you have multiple Azure subscriptions and want to use a specific one, set the active subscription using the `az account set` command.

   ```bash
   az account set --subscription <subscription-id>
   ```

   Replace `<subscription-id>` with the ID of the subscription you want to use.

3. **Create Resource Group**: If you haven't already created a resource group for your AKS cluster, you can create one using the `az group create` command.

   ```bash
   az group create --name <resource-group-name> --location <location>
   ```
   #### Example
   ```bash
   az group create --name aksgroup --location eastus
   ```

   Replace `<resource-group-name>` with the name of your resource group and `<location>` with the Azure region where you want to deploy the resources.

5. **Install AKS Cluster**: Now, you can create the AKS cluster using the `az aks create` command. Here's an example command:

   ```bash
   az aks create --resource-group <resource-group-name> --name <cluster-name> --node-count <node-count> --node-vm-size <vm-size> --enable-addons monitoring --generate-ssh-keys
   ```

   - Replace `<resource-group-name>` with the name of the resource group you created.
   - Replace `<cluster-name>` with the name you want to give to your AKS cluster.
   - `<node-count>` specifies the number of nodes in the cluster.
   - `<vm-size>` specifies the size of the VMs for the cluster nodes (e.g., Standard_DS2_v2).
   - `--enable-addons monitoring` enables Azure Monitor for container monitoring.
   - `--generate-ssh-keys` generates SSH public and private key files for node access.
#### Example
```bash
az aks create -n akscluster -g aksgroup --node-count 2 --generate-ssh-keys
aks list --resource-group aksgroup --output table
```


6. **Get Credentials**: After the cluster is created, you need to get the credentials to access the cluster using the `kubectl` command-line tool. Use the following command:

   ```bash
   az aks get-credentials --resource-group <resource-group-name> --name <cluster-name>
   ```

   Replace `<resource-group-name>` and `<cluster-name>` with the corresponding values you used during cluster creation.

7. **Verify Cluster**: You can verify that the cluster is running and accessible by running the `kubectl` command to get the cluster nodes:

   ```bash
   kubectl get nodes
   ```

### Uninstallation:

To uninstall the AKS cluster and associated resources, you can delete the resource group containing the AKS cluster. Use the following command:

```bash
az group delete --name <resource-group-name> 
```

Replace `<resource-group-name>` with the name of the resource group containing your AKS cluster.

This command will delete the resource group and all resources (including the AKS cluster) contained within it. Make sure to verify that you want to delete these resources, as this action cannot be undone.

