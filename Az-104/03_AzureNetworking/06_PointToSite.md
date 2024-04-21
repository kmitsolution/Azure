## Point-to-Site(P2S) VPN in Azure

A Point-to-Site (P2S) VPN in Azure is a connection between a single device and an Azure virtual network (VNet). It enables remote users to securely connect to the Azure VNet over the internet. Here's a brief overview of how it works:

**Connect Local Laptop to VNET (its Private IP)**

<img width="827" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/d18f12b7-0e18-449e-bc7f-fd251d2c9537">


**Address Pool**
  In Azure, a Point-to-Site (P2S) VPN connection is a secure connection between an individual client computer and Azure. This connection is useful when you need to establish a secure connection from an individual client computer to resources within an Azure Virtual Network (VNet). 

The Address Pool in a Point-to-Site configuration refers to the range of IP addresses that Azure assigns to the VPN clients when they connect to the Azure VNet. This pool of addresses should be unique and not overlap with any existing IP ranges within the Azure VNet or your on-premises network. 

When configuring a P2S VPN connection in Azure, you specify the address pool range as part of the configuration. Azure dynamically assigns an IP address from this pool to each VPN client when they establish a connection. This ensures that each client has a unique IP address within the virtual network.

Managing the address pool effectively is important to ensure that you have enough available IP addresses for all potential VPN clients without causing conflicts with other resources in your network. Azure provides tools to help monitor and manage the address pool as needed.

**P2S Protocol**

**OpenVPN® Protocol**, an SSL/TLS based VPN protocol. A TLS VPN solution can penetrate firewalls, since most firewalls open TCP port 443 outbound, which TLS uses. OpenVPN can be used to connect from Android, iOS (versions 11.0 and above), Windows, Linux, and Mac devices (macOS versions 10.13 and above).

**Secure Socket Tunneling Protocol (SSTP)**, a proprietary TLS-based VPN protocol. A TLS VPN solution can penetrate firewalls, since most firewalls open TCP port 443 outbound, which TLS uses. SSTP is only supported on Windows devices. Azure supports all versions of Windows that have SSTP and support TLS 1.2 (Windows 8.1 and later).

**IKEv2 VPN**, a standards-based IPsec VPN solution. IKEv2 VPN can be used to connect from Mac devices (macOS versions 10.11 and above).

<img width="765" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/9f7452cb-22d5-43d1-819c-42c4edbd0e28">

1. Generate Self signed Certificate using <a href=https://learn.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-certificates-point-to-site> Self Signed Certificate </a>
2. Once certificates are installed Export them to some folder (Public key without private key and Private key with some password)
3. Put root certificate using public key in P2S configuration(without first and last line)
4. Install Client .pfx file on the client machine
5. Download VPN Client using P2S configuration Page.
6. Once VPN Client is installed then Check in the VPN Settings and Install VPN Client 
<br/>

1. **Azure Virtual Network (VNet)**: This is your isolated network environment in Azure. You create and configure your VNet with subnets, IP address ranges, and other settings as needed.

2. **Gateway Subnet**: Within your VNet, you create a subnet specifically for the VPN gateway. This subnet hosts the VPN gateway resources that facilitate the secure connection.

3. **VPN Gateway**: Azure provides a VPN gateway service that acts as the bridge between your Azure VNet and the external devices connecting to it. You configure this gateway to support Point-to-Site connections.

4. **Client Setup**: On the client side (the device connecting to Azure), you install a VPN client software provided by Azure. This client software establishes a secure connection to the Azure VPN gateway.

5. **Authentication**: Azure supports various authentication methods for P2S VPN connections, including certificates, Azure Active Directory, and RADIUS authentication. You configure the authentication method based on your security requirements.

6. **Secure Connection**: Once the client software is configured and the authentication is successful, the device establishes a secure VPN tunnel to the Azure VNet. This tunnel encrypts all traffic between the client device and the Azure VNet, ensuring data privacy and integrity.

7. **Access Control**: After connecting to the Azure VNet, remote users can access resources within the VNet, such as virtual machines, databases, and other services. Access control can be further managed through network security groups (NSGs) and other Azure security features.

Overall, a Point-to-Site VPN in Azure provides a secure and convenient way for remote users to access resources hosted in Azure VNet, extending the reach of your network infrastructure to external devices without compromising security.
