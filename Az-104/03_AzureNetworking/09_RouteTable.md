# Route Table in Azure
In Azure, a Route Table is a logical construct used in networking to control and define the routing behavior for network traffic within a virtual network (VNet) or between virtual networks. 

<img width="900" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/82f071d1-9360-49cc-91cb-6000bba3f95d">

Here's an overview of how Route Tables work in Azure:

1. **Definition**: A Route Table is essentially a set of rules (routes) that determine where network traffic should be directed. Each route specifies a destination and the next hop for packets destined for that destination.

2. **Virtual Network Scope**: Route Tables are associated with a specific virtual network and are applicable to all the subnets within that virtual network.

3. **Default Route**: By default, every virtual network in Azure comes with a system-generated Route Table. This system-generated Route Table contains a default route that directs all outbound traffic to the internet through the Azure Internet Gateway.

4. **Custom Routes**: You can customize the Route Table by adding custom routes to override the default behavior. These custom routes can specify different next-hop destinations for specific IP ranges or traffic types.

5. **Effective Routes**: The effective route table is the combination of all applicable route tables (system-generated and custom) for a subnet. When determining the next hop for a packet, Azure evaluates the effective route table for the subnet associated with the traffic.

6. **Associations**: Route Tables are associated with subnets within a virtual network. You can associate a single Route Table with multiple subnets, or each subnet can have its own associated Route Table.

7. **Priority**: When multiple routes match a destination, Azure uses the most specific route (longest prefix match) with the highest priority. If there are multiple routes with the same prefix length, Azure uses the route with the lowest numeric route priority.

```
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters" -Name IpEnableRouter -Value 1

```
Route Tables are a fundamental component of Azure networking, providing flexibility and control over how traffic is routed within virtual networks and between virtual networks and the internet. They play a crucial role in designing and managing network architecture in Azure environments.
