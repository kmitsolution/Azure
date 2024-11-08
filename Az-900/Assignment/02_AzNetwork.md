### **Azure Assignment 1: VNet Peering, VM Setup, and Network Configuration**

**Objective:**  
In this assignment, you are tasked with creating two virtual networks (VNet), one for each client (Client-1 and Client-2), and setting up communication between the two using VPN Peering. You will also configure Ubuntu virtual machines in each network, assign static private and public IPs, and verify that the VMs can communicate with each other across VNets.

---

### **Assignment Tasks:**

#### **Part 1: VNet and Subnet Creation**

1. **Create Two Virtual Networks:**
   - **Client-1 VNet**  
     - Name: `vnet-client1`  
     - Region: East US  
     - Address Space: `10.0.0.0/16`
   
   - **Client-2 VNet**  
     - Name: `vnet-client2`  
     - Region: East US  
     - Address Space: `10.1.0.0/16`

2. **Create Subnets for Each VNet:**
   - For **Client-1 VNet**, create the following subnets:  
     - Subnet 1: `subnet-client1` (Address Space: `10.0.0.0/24`)
   
   - For **Client-2 VNet**, create the following subnets:  
     - Subnet 1: `subnet-client2` (Address Space: `10.1.0.0/24`)

---

#### **Part 2: Network Security Groups (NSG) Configuration**

1. **Create Network Security Groups:**
   - For **Client-1 VNet**, create an NSG named `nsg-client1` and configure it to allow all inbound and outbound traffic (open all ports).  
     - Inbound Rule: Allow `Any` source, `Any` destination, `Any` port, `Any` protocol.  
     - Outbound Rule: Allow `Any` source, `Any` destination, `Any` port, `Any` protocol.
   
   - For **Client-2 VNet**, create an NSG named `nsg-client2` and configure it to allow all inbound and outbound traffic (open all ports).  
     - Inbound Rule: Allow `Any` source, `Any` destination, `Any` port, `Any` protocol.  
     - Outbound Rule: Allow `Any` source, `Any` destination, `Any` port, `Any` protocol.

2. **Associate NSG with respective subnets:**  
   - Associate `nsg-client1` with `subnet-client1`.  
   - Associate `nsg-client2` with `subnet-client2`.

---

#### **Part 3: Create Ubuntu Virtual Machines in Each VNet**

1. **Create an Ubuntu VM in Client-1 VNet:**
   - VM Name: `vm-client1`
   - OS: Ubuntu Server 20.04 LTS
   - Size: Standard B1s (or any low-cost VM size)
   - Public IP: Dynamic
   - Private IP: Static (`10.0.0.4`)

2. **Create an Ubuntu VM in Client-2 VNet:**
   - VM Name: `vm-client2`
   - OS: Ubuntu Server 20.04 LTS
   - Size: Standard B1s (or any low-cost VM size)
   - Public IP: Dynamic
   - Private IP: Static (`10.1.0.4`)

3. **Verify VM creation:**
   - After the VMs are created, ensure that both VMs have the correct public IP addresses assigned, and static private IPs are configured correctly.
   - Use Azure Portal or Azure CLI to check the VM status and IP configurations.

---

#### **Part 4: Set Up VPN Peering Between VNets**

1. **Create VPN Gateway for Each VNet:**
   - For **Client-1 VNet**: Create a VPN Gateway (in the `vnet-client1` region) named `vpn-gateway-client1`.
   - For **Client-2 VNet**: Create a VPN Gateway (in the `vnet-client2` region) named `vpn-gateway-client2`.

2. **Configure VPN Peering:**
   - **Client-1 VNet Peering Configuration:**  
     - Name the peering `client1-to-client2-peering`  
     - Peer with `vnet-client2`.  
     - Set up **Allow Gateway Transit** on `vnet-client1`.
   
   - **Client-2 VNet Peering Configuration:**  
     - Name the peering `client2-to-client1-peering`  
     - Peer with `vnet-client1`.  
     - Set up **Use Remote Gateway** on `vnet-client2`.

3. **Verify VPN Connectivity:**
   - Once peering is set up, verify the VPN tunnel is active by checking the status in the Azure Portal.  
   - Use the **Network Watcher** tool to verify the connection and ensure both VNets can route traffic between each other.

---

#### **Part 5: Verification**

1. **Test VM Connectivity:**
   - From the **Ubuntu VM in Client-1** (`vm-client1`), run the following command to verify communication with the VM in Client-2:
     ```bash
     ping 10.1.0.4
     ```
   - From the **Ubuntu VM in Client-2** (`vm-client2`), run the following command to verify communication with the VM in Client-1:
     ```bash
     ping 10.0.0.4
     ```

2. **Verify successful ping responses:**  
   - If the ping responses are successful from both VMs, the VPN peering and routing are configured correctly.

---

### **Assignment 2: Subnet Traffic Routing Using DMZ**

**Objective:**  
This assignment involves creating three subnets: Public, Private, and DMZ. You will configure traffic routing such that all traffic from VMs in the Public subnet travels via the DMZ subnet.

---

### **Assignment Tasks:**

#### **Part 1: VNet and Subnet Creation**

1. **Create a Virtual Network (VNet):**
   - Name: `vnet-client`
   - Region: East US
   - Address Space: `10.0.0.0/16`

2. **Create Three Subnets:**
   - **Public Subnet:** `10.0.1.0/24` (Address space for public-facing resources)
   - **Private Subnet:** `10.0.2.0/24` (Address space for internal, non-exposed resources)
   - **DMZ Subnet:** `10.0.3.0/24` (Address space for the security layer)

#### **Part 2: Route Traffic via DMZ Subnet**

1. **Create Route Table:**
   - Create a **Route Table** named `route-table-dmz`.
   - Add a route in the route table to direct traffic from the Public subnet to the DMZ subnet. For example:
     - Destination: `0.0.0.0/0`  
     - Next Hop: **Virtual Appliance** (You can use an NVA like a firewall or routing device in the DMZ subnet).

2. **Associate Route Table with Public Subnet:**
   - Associate the `route-table-dmz` with the **Public Subnet** (`10.0.1.0/24`).

#### **Part 3: Create Virtual Machines**

1. **Create a VM in Public Subnet:**
   - VM Name: `vm-public`
   - OS: Ubuntu Server 20.04 LTS
   - IP Configuration: Assign a dynamic Public IP and Static Private IP (`10.0.1.4`).

2. **Create a VM in Private Subnet:**
   - VM Name: `vm-private`
   - OS: Ubuntu Server 20.04 LTS
   - Private IP: `10.0.2.4` (Static)

3. **Create a VM in DMZ Subnet:**
   - VM Name: `vm-dmz`
   - OS: Ubuntu Server 20.04 LTS
   - Private IP: `10.0.3.4` (Static)

#### **Part 4: Verification**

1. **Test Traffic Routing:**
   - From the **Public VM** (`vm-public`), ping the **Private VM** (`vm-private`) and confirm that the traffic flows through the DMZ subnet.
   
   - Use **Traceroute** or **Network Watcher** to ensure the traffic from the Public VM (`vm-public`) routes through the DMZ subnet (`vm-dmz`).

   - Example command to verify:
     ```bash
     traceroute 10.0.2.4  # To test if traffic goes through the DMZ
     ```

2. **Verify Successful Ping:**
   - Ensure that all traffic from the Public subnet goes through the DMZ and reaches the Private subnet.

---

### **Submission Requirements:**
- Detailed screenshots of the VNet, subnet, and NSG configurations.
- VM creation details, including public and private IPs.
- VPN peering configuration and verification results.
- Results from the ping and traceroute tests, proving successful VM communication across VNets and proper routing via DMZ.

