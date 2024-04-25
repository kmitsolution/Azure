# Standard Load Balancer in Azure

In Azure, the Standard Load Balancer is a highly available and scalable layer 4 (TCP, UDP) load balancer that distributes incoming network traffic across multiple virtual machines (VMs) in Azure. It offers several features and benefits:

A <b>public load balancer</b> can provide outbound connections for virtual machines (VMs) inside your virtual network. These connections are accomplished by translating their private IP addresses to public IP addresses. Public Load Balancers are used to load balance internet traffic to your VMs.

An <b>internal (or private) load balancer </b>is used where private IPs are needed at the frontend only. Internal load balancers are used to load balance traffic inside a virtual network. A load balancer frontend can be accessed from an on-premises network in a hybrid scenario.

![image](https://github.com/kmitsolution/Azure/assets/84008107/88a2da03-afc0-4bf0-9647-a10f861ee4cd)

1. **High Availability**: Standard Load Balancer provides redundancy and high availability for your applications by distributing incoming traffic across multiple VM instances within an Azure Virtual Network (VNet) or across VNets in the same region.

2. **Scalability**: It scales with your application's traffic demands, allowing you to handle increasing loads without manual intervention. You can dynamically add or remove VM instances from the load balancer pool as needed.

3. **Health Probes**: Standard Load Balancer continuously monitors the health of your VM instances by sending health probes to each instance. If an instance fails health probes, it is automatically removed from the load balancer rotation until it becomes healthy again.

4. **Session Persistence**: It supports session persistence (also known as sticky sessions), ensuring that incoming requests from the same client are consistently routed to the same backend VM instance, which is essential for maintaining session state in stateful applications.

5. **Backend Pool Configuration**: You can configure backend pools to distribute traffic to VMs based on various criteria such as round-robin, least connections, or specific rules defined by you.

6. **Security**: Standard Load Balancer supports integration with Azure Firewall and Network Security Groups (NSGs) to enforce network security policies and protect your applications from unauthorized access.

7. **Cross-zone Load Balancing**: It supports distributing traffic evenly across VM instances located in different availability zones within the same region, increasing fault tolerance and availability.

8. **Outbound Connections**: Standard Load Balancer can be used for outbound connections, allowing VM instances to access the internet while using the same public IP address for all outbound traffic.

9. **Monitoring and Logging**: Azure provides monitoring and logging capabilities for Standard Load Balancer through Azure Monitor and Azure Log Analytics, allowing you to track performance, troubleshoot issues, and gain insights into your application's traffic patterns.

Overall, Azure Standard Load Balancer is a powerful tool for building highly available and scalable applications in the Azure cloud environment.
