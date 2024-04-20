# Azure Networking

In Azure, networking revolves around the concept of Virtual Networks (VNets), Subnets, and Network Interface Cards (NICs). Let's break down each of these components:

<img width="940" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/af7d0cc7-b20a-416f-91b2-8b5259f6cc63">



1. **Virtual Network (VNet)**:
   - A Virtual Network (VNet) in Azure is a logically isolated network that you create in the cloud. It provides a way to connect Azure resources securely, much like a traditional network in your on-premises datacenter.
   - VNets enable you to control IP address ranges, DNS settings, security policies, and route tables.
   - When you create resources in Azure, such as virtual machines or Azure App Services, you can deploy them within a VNet to provide secure communication between resources.

2. **Subnet**:
   - Subnets are subdivisions of a VNet and are used to organize and segment network traffic within the VNet.
   - They allow you to group resources based on their function, security requirements, or other criteria.
   - Each subnet within a VNet can have its own IP address range, and you can define network security policies at the subnet level using Network Security Groups (NSGs).

3. **Network Interface Card (NIC)**:
   - A Network Interface Card (NIC) in Azure is a virtual representation of a physical network adapter.
   - When you create a virtual machine (VM) or other resources that need network connectivity, Azure automatically creates a NIC and attaches it to the resource.
   - NICs provide connectivity to the VNet and enable communication with other resources within the VNet or over the internet.
   - You can configure properties of the NIC such as IP address, subnet, network security groups, and IP forwarding.

In summary, in Azure networking:
- **Virtual Networks (VNets)** provide a way to connect and isolate Azure resources logically.
- **Subnets** are used to segment and organize network traffic within a VNet.
- **Network Interface Cards (NICs)** provide network connectivity to Azure resources and are attached to resources such as virtual machines.

Understanding these components is crucial for designing and managing Azure networking solutions effectively, ensuring secure and efficient communication between Azure resources.
