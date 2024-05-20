## AKS Properties

When creating an AKS (Azure Kubernetes Service) cluster in Azure, you can specify both the Azure Resource Group and the Azure Kubernetes Service Managed Resource Group. Here's how they are related:

<img width="927" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/c5090f5a-bb9b-4173-ba9c-5a8b8562beb0">


1. **Azure Resource Group**: An Azure Resource Group is a logical container that holds related resources for an Azure solution. This group includes all the resources you deploy for your solution, such as virtual machines, storage accounts, virtual networks, databases, and AKS clusters. Organizing resources into resource groups makes it easier to manage and monitor them collectively.

2. **Azure Kubernetes Service Managed Resource Group**: When you create an AKS cluster, Azure automatically creates a separate resource group, known as the Azure Kubernetes Service Managed Resource Group, to manage the underlying resources specific to AKS. This includes resources like virtual machines, virtual networks, storage accounts, network security groups, and more, which are provisioned to support the Kubernetes cluster.

Here's how they are related during the creation of an AKS cluster:

- During the creation process, you specify the Azure Resource Group where you want to deploy the AKS cluster. This is the resource group that you manage and where you can find your AKS cluster along with other resources related to your solution.
  
- Within this specified Azure Resource Group, Azure creates another resource group, the Azure Kubernetes Service Managed Resource Group, to manage the AKS-specific resources. This managed resource group is created and managed by Azure and contains resources necessary for the operation of the AKS cluster.

By separating AKS-specific resources into its managed resource group, Azure ensures better management and isolation of resources, simplifying operations and improving security and governance for your Kubernetes clusters.

Ah, I see! When you create an AKS cluster in Azure, you can specify various configurations, including the pod CIDR range, service CIDR range, Azure Resource Group, Kubernetes version, and node pools. Here's a breakdown of each:

1. **Pod CIDR Range**: This is the range of IP addresses assigned to pods (containers) running on your AKS cluster. It's important to ensure that this CIDR range does not overlap with your on-premises network or any other networks to which your AKS cluster needs to communicate.

2. **Service CIDR Range**: This is the range of IP addresses assigned to Kubernetes services in your AKS cluster. Services are accessed by other pods within the cluster and can also be exposed externally.

3. **Azure Resource Group**: When you create an AKS cluster, you specify the Azure Resource Group in which the cluster will be deployed. Resource groups are logical containers for grouping Azure resources, making it easier to manage and organize them.

4. **Configuration Kubernetes Version**: You can specify the version of Kubernetes you want to use for your AKS cluster. Azure provides a list of supported Kubernetes versions, and you can choose the one that best fits your requirements. It's important to keep your cluster up to date with the latest Kubernetes versions for security patches and new features.

5. **Node Pools**: Node pools in AKS allow you to create groups of nodes with specific configurations, such as VM size, OS disk type, and Kubernetes version. Each node pool can be scaled independently, allowing you to optimize resources and performance for different workloads within your cluster.

Configuring these parameters appropriately during AKS cluster creation ensures that your cluster is tailored to meet your specific requirements in terms of networking, scalability, security, and compatibility with your applications.
