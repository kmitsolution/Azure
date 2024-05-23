## AKS Preview Extension and Registering Feature
Azure CLI to manage Azure Kubernetes Service (AKS) and its preview features. Let's break down the steps :

1. **Check if AKS preview extension is installed**:
   ```powershell
   az extension list-available -o table | Select-String -Pattern aks-preview
   ```

2. **Add AKS preview extension** (if not installed):
   ```powershell
   az extension add --name aks-preview --allow-preview
   ```

3. **Update the AKS preview extension** (if already installed):
   ```powershell
   az extension update --name aks-preview
   ```

4. **Check the state of the AKS-VPAPreview feature**:
   ```powershell
   az feature show --namespace "Microsoft.ContainerService" --name "AKS-VPAPreview"
   ```

5. **If the feature is not registered, register it**:
   ```powershell
   az feature register --namespace "Microsoft.ContainerService" --name "AKS-VPAPreview"
   ```

6. **After registering the feature, ensure the provider is registered**:
   ```powershell
   az provider register --namespace Microsoft.ContainerService
   ```

Regarding documentation, you can find detailed information about Azure Kubernetes Service (AKS) and its preview features on the official Azure documentation website. You can start with the documentation for AKS and then navigate to specific topics related to preview features or updates.
