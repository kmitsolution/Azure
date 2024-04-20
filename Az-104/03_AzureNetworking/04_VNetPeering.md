# Azure VNET Peering
Virtual Network (VNet) peering is a networking feature in Azure that enables you to connect two VNets in the same Azure region through the Azure backbone network. This allows resources in the peered VNets to communicate with each other as if they were on the same network. VNet peering is commonly used to create network topologies with distributed applications, services, or workloads while maintaining isolation and segmentation between different environments or components.

Here's how VNet peering works and its key characteristics:

<img width="902" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/7a1e4873-4db6-4008-bb0e-7d728f927389">


1. **Peer VNets**:
   - With VNet peering, you can establish a peering relationship between two VNets. This relationship is bidirectional, allowing resources in both VNets to communicate with each other.
   - Peering can be established between VNets in the same Azure region, regardless of whether they belong to the same subscription or different subscriptions.

2. **Private Connectivity**:
   - VNet peering enables private connectivity between resources in the peered VNets. Traffic between peered VNets stays within the Azure backbone network and does not traverse the public internet.
   - This ensures low-latency, high-speed communication between resources and enhances security by keeping traffic isolated from the public internet.

3. **Traffic Flow**:
   - When resources in one VNet need to communicate with resources in the other VNet, Azure routes the traffic through the peering connection.
   - Traffic between peered VNets is routed based on the private IP addresses of the resources, allowing for seamless communication using standard IP protocols.

4. **Transitive Peering**:
   - VNet peering supports transitive connectivity, meaning that if VNet A is peered with VNet B, and VNet B is peered with VNet C, then resources in VNet A can communicate with resources in VNet C through the transitive peering relationship.
   - This enables you to create complex network topologies with multiple interconnected VNets while maintaining isolation and segmentation between different components.

5. **Limitations**:
   - While VNet peering offers many benefits, it's important to be aware of its limitations, such as:
     - Peering is not transitive across global Azure regions.
     - There are limits on the number of peering relationships per VNet and the number of peered VNets per subscription.

Overall, VNet peering is a powerful networking feature in Azure that simplifies network connectivity and enables you to build scalable and resilient architectures with distributed resources while maintaining isolation and security.
