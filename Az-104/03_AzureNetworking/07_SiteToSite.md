# Site-to-Site VPN

<img width="755" alt="image" src="https://github.com/kmitsolution/Azure/assets/84008107/86304ab4-01dd-445f-a11c-5b7e7f4721b6">

Setting up a site-to-site VPN (Virtual Private Network) connection in Azure allows you to securely connect your on-premises network to your Azure virtual network over the internet. This enables seamless communication between resources hosted in Azure and those in your local network. Here's a general guide on how to set up a site-to-site VPN in Azure:

1. **Prepare Your Azure Environment**:
   - Log in to the Azure portal (https://portal.azure.com).
   - Navigate to the Azure Virtual Network service.
   - Create or select the virtual network to which you want to connect your on-premises network.

2. **Create a Virtual Network Gateway**:
   - In the Azure portal, navigate to the virtual network.
   - Click on "Create Gateway" and choose "Virtual network gateway".
   - Specify the gateway details including name, VPN type (Route-based or Policy-based), SKU, and virtual network.

3. **Configure Local Network Gateway**:
   - Create a Local Network Gateway object in Azure to represent your on-premises network.
   - Specify the public IP address of your on-premises VPN device, the address space of your on-premises network, and other relevant details.

4. **Configure VPN Device**:
   - Configure your on-premises VPN device to establish a VPN connection with Azure. The configuration steps depend on the specific VPN device you're using.
   - You'll need to specify the public IP address of the Azure Virtual Network Gateway, configure the IPsec/IKE parameters, and define the local and remote network settings.

5. **Establish VPN Connection**:
   - Once the configurations are done on both ends, initiate the connection from your on-premises VPN device.
   - Monitor the connection status in the Azure portal to ensure it establishes successfully.

6. **Configure Routing**:
   - Configure the routing tables in both Azure and your on-premises network to ensure that traffic destined for each network is routed correctly through the VPN connection.

7. **Test Connectivity**:
   - Test connectivity between resources in Azure and your on-premises network to verify that the VPN connection is working as expected.
   - Troubleshoot any connectivity issues if necessary.

8. **Monitor and Maintain**:
   - Regularly monitor the VPN connection status and performance in the Azure portal.
   - Keep configurations up to date and perform maintenance tasks as needed.

Remember to follow security best practices when setting up and configuring your VPN connection to ensure the confidentiality, integrity, and availability of your network traffic.
