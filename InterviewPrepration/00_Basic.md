
---

## 1️⃣ Azure Networking

### **Q1. What is an Azure Virtual Network (VNet)?**

**Answer:**
An Azure Virtual Network is a logically isolated network in Azure that allows Azure resources like virtual machines, App Services, and databases to communicate securely with each other, the internet, and on-premises networks.

---

### **Q2. What is the difference between a subnet and a VNet?**

**Answer:**
A VNet is a virtual network boundary in Azure, while a subnet is a smaller network segment inside a VNet used to organize and isolate resources.

---

### **Q3. What is VNet peering?**

**Answer:**
VNet peering allows two VNets to communicate with each other privately using Azure’s backbone network without using the public internet.

---

### **Q4. What is the difference between a public endpoint and a private endpoint?**

**Answer:**
A public endpoint exposes a service to the internet, whereas a private endpoint provides private connectivity to an Azure service using a private IP inside a VNet.

---

## 2️⃣ Network Security Group (NSG)

### **Q5. What is an Azure Network Security Group (NSG)?**

**Answer:**
An NSG is a security feature that controls inbound and outbound network traffic to Azure resources using allow and deny rules based on IP address, port, and protocol.

---

### **Q6. Where can an NSG be applied?**

**Answer:**
An NSG can be associated with:

* A subnet
* A network interface (NIC) of a virtual machine

---

### **Q7. What is the difference between NSG and Azure Firewall?**

**Answer:**
NSG provides basic traffic filtering at the network level, while Azure Firewall is a fully managed, stateful firewall with advanced security features and centralized control.

---

## 3️⃣ Network Interface (NIC)

### **Q8. What is a Network Interface (NIC) in Azure?**

**Answer:**
A NIC is a component that connects a virtual machine to an Azure Virtual Network and allows the VM to communicate over the network.

---

### **Q9. Can a VM have multiple NICs?**

**Answer:**
Yes, Azure supports multiple NICs per VM depending on the VM size, which is useful for network separation or advanced networking scenarios.

---

## 4️⃣ Microsoft Entra ID (Azure AD)

### **Q10. What is Microsoft Entra ID?**

**Answer:**
Microsoft Entra ID is Azure’s cloud-based identity and access management service that helps manage users, groups, and application access securely.

---

### **Q11. What is Single Sign-On (SSO)?**

**Answer:**
SSO allows users to sign in once and access multiple applications without needing to log in again.

---

### **Q12. What is Conditional Access in Entra ID?**

**Answer:**
Conditional Access enforces access controls based on conditions such as user location, device compliance, or risk level.

---

### **Q13. What is the difference between Entra ID and Entra Domain Services?**

**Answer:**
Entra ID is a cloud identity service, while Entra Domain Services provides traditional domain features like LDAP and Group Policy without managing domain controllers.

---

## 5️⃣ Azure App Services

### **Q14. What is Azure App Service?**

**Answer:**
Azure App Service is a Platform as a Service (PaaS) offering used to host web applications, REST APIs, and mobile backends without managing servers.

---

### **Q15. When should you use App Service instead of a Virtual Machine?**

**Answer:**
Use App Service when you want to deploy applications quickly without worrying about OS management, patching, or infrastructure maintenance.

---

## 6️⃣ Virtual Machine Scale Sets (VMSS)

### **Q16. What is Azure Virtual Machine Scale Set (VMSS)?**

**Answer:**
VMSS allows you to deploy and manage a group of identical virtual machines with automatic scaling and high availability.

---

### **Q17. How does VMSS improve availability?**

**Answer:**
VMSS distributes VMs across availability zones or fault domains and automatically replaces unhealthy instances.

---

## 7️⃣ Azure Container Registry (ACR)

### **Q18. What is Azure Container Registry (ACR)?**

**Answer:**
ACR is a private container image registry used to store and manage Docker and OCI container images in Azure.

---

### **Q19. Why is ACR used with AKS?**

**Answer:**
ACR securely stores container images that AKS pulls to deploy containerized applications.

---

## 8️⃣ Azure Kubernetes Service (AKS)

### **Q20. What is Azure Kubernetes Service (AKS)?**

**Answer:**
AKS is a managed Kubernetes service that simplifies deploying, scaling, and managing containerized applications.

---

### **Q21. What is the difference between AKS and App Service?**

**Answer:**
App Service is best for simple web applications, while AKS is used for complex, microservices-based, containerized applications requiring orchestration.
