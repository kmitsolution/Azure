# Subnet Reserved IP Addresses

In a subnet with the CIDR notation "10.0.1.0/24", you're correct that theoretically, there should be 256 IP addresses available for assignment. However, in reality, not all IP addresses within a subnet are available for use by resources. Here's why you might see fewer than 256 available IP addresses:
<img width="710" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/0636149c-6cf8-4663-98c6-a42b6b660a68">

1. **Reserved Addresses**:
   - In Azure, certain IP addresses within a subnet are reserved for specific purposes and cannot be assigned to resources. These reserved addresses include:
     - The first IP address in the subnet (e.g., 10.0.1.0) is the network address and cannot be assigned to resources.
     - The last IP address in the subnet (e.g., 10.0.1.255) is the broadcast address and cannot be assigned to resources.
     - Typically, Azure reserves five IP addresses within each subnet for system use, such as Azure services and network infrastructure. These addresses are not available for assignment to user resources.

2. **Gateway Addresses**:
   - If you have gateway resources deployed within the subnet, such as Azure VPN Gateway or Azure ExpressRoute Gateway, additional IP addresses may be reserved for these gateways.

3. **Network Overhead**:
   - Azure may reserve additional IP addresses for network overhead and management purposes, such as DHCP (Dynamic Host Configuration Protocol) leases, DNS (Domain Name System) services, and network infrastructure.

Taking these factors into account, the actual number of available IP addresses for assignment to user resources may be fewer than the total number of addresses in the subnet's address space. In the case of your example subnet "10.0.1.0/24", seeing 251 available IP addresses instead of 256 is likely due to the combination of reserved addresses, gateway addresses, and network overhead.
