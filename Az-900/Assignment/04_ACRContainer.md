To complete this assignment without using Azure CLI, we will perform all tasks via the **Azure Portal**. This will involve several key steps, including:

1. **Creating a Virtual Machine (VM)** and installing Docker on it.
2. **Downloading the NGINX Docker image** on the VM.
3. **Creating an Azure Container Registry (ACR)**.
4. **Pushing the NGINX image to the ACR repository**.
5. **Creating an Azure Container Instance (ACI)** from the image stored in ACR.

Let’s go through each of these steps one by one.

---

### **Step 1: Create a Virtual Machine (VM) in Azure**

1. **Go to the Azure Portal**:
   - Open [Azure Portal](https://portal.azure.com).

2. **Create a new Virtual Machine**:
   - In the left sidebar, click on **"Create a resource"**.
   - Search for **"Virtual Machine"** and select it.
   - Click **"Create"**.

3. **Configure VM settings**:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Create a new resource group or use an existing one (e.g., `MyResourceGroup`).
   - **Virtual Machine Name**: Enter a name for the VM, e.g., `MyVM`.
   - **Region**: Select a region, e.g., **East US**.
   - **Image**: Choose **Ubuntu 20.04 LTS** (or any Linux distribution that supports Docker).
   - **Size**: Select a size (e.g., `Standard B1s`).
   - **Authentication Type**: Use **SSH public key** for secure login. You can also use **Password** for simplicity, but SSH is recommended for production.
     - For SSH: Provide a public SSH key or generate one if you haven't already.
   - **Inbound Port Rules**: Allow SSH (port 22) for SSH access.

4. **Review and create the VM**:
   - Click **Next: Disks**. You can leave the default settings.
   - Click **Next: Networking** and configure the network (default settings are fine).
   - Click **Review + Create** and then click **Create** to provision the VM.

---

### **Step 2: Connect to the VM and Install Docker**

1. **Connect to the VM**:
   - Once the VM is created, go to the **Overview** page of the VM.
   - Click **Connect** and select **SSH** to get the SSH command.
   - Open your terminal (or use a tool like **Putty** if you use Windows) and connect using the provided SSH command.

   Example:
   ```bash
   ssh azureuser@<your-vm-ip>
   ```

2. **Install Docker on the VM**:
   Once connected to the VM, update the package list and install Docker with the following commands:

   ```bash
   sudo apt-get update
   sudo apt-get install -y docker.io
   ```

3. **Start and enable Docker**:
   Run these commands to ensure Docker starts on boot:

   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

4. **Verify Docker installation**:
   Check if Docker is installed correctly by running:

   ```bash
   docker --version
   ```

---

### **Step 3: Download the NGINX Docker Image**

1. **Pull the NGINX Docker image**:
   Run the following command to pull the official NGINX image from Docker Hub:

   ```bash
   sudo docker pull nginx
   ```

2. **Verify the image is downloaded**:
   You can check the available Docker images by running:

   ```bash
   sudo docker images
   ```

---

### **Step 4: Create an Azure Container Registry (ACR)**

1. **Create ACR in the Azure Portal**:
   - In the Azure Portal, go to **"Create a resource"**.
   - Search for **"Container Registry"** and select it.
   - Click **"Create"**.

2. **Configure the ACR**:
   - **Subscription**: Select your subscription.
   - **Resource Group**: Use the same resource group as the VM, or create a new one.
   - **Registry Name**: Enter a unique name for your ACR, e.g., `mynginxacr`.
   - **Location**: Select the region where your VM is deployed.
   - **SKU**: Choose **Basic** (for small-scale testing).
   - **Admin user**: Turn this on, which allows you to authenticate to the ACR using a username and password.

3. **Review and create**:
   - Click **Review + Create** and then **Create**.

---

### **Step 5: Push the NGINX Image to ACR**

1. **Login to the ACR**:
   - In the **Azure Portal**, navigate to your ACR and click on **Access keys** under **Settings**.
   - Copy the **Login Server** URL and **Username** and **Password** from here.
   
2. **Tag the NGINX image**:
   On the VM, tag the pulled NGINX image with your ACR login server URL:

   ```bash
   sudo docker tag nginx <acr-login-server>/<repository-name>:latest
   ```

   Replace `<acr-login-server>` with the Login Server URL from the ACR (e.g., `mynginxacr.azurecr.io`), and `<repository-name>` with the name of the repository (e.g., `nginx-repo`).

3. **Login to ACR using Docker**:
   Run the following command and enter the credentials (username and password) when prompted:

   ```bash
   sudo docker login <acr-login-server>
   ```

4. **Push the image to ACR**:
   Push the tagged image to the ACR:

   ```bash
   sudo docker push <acr-login-server>/<repository-name>:latest
   ```

---

### **Step 6: Create an Azure Container Instance (ACI) from the Image in ACR**

1. **Go to Azure Container Instances**:
   - In the Azure Portal, search for **"Container Instances"** and select it.
   - Click on **"Create"** to create a new container instance.

2. **Configure ACI settings**:
   - **Subscription**: Choose your subscription.
   - **Resource Group**: Use the same resource group.
   - **Container Name**: Give a name to the container (e.g., `nginx-container`).
   - **Region**: Choose the same region as the VM and ACR.
   - **Image**: Under **Image Source**, select **Azure Container Registry** and then choose your ACR and the `nginx-repo` image you pushed earlier.
   - **Ports**: Expose port 80, as NGINX typically runs on this port.

3. **Review and Create**:
   - Click **Review + Create** and then **Create** to provision the container.

---

### **Step 7: Verify the Container Instance**

1. **Access the NGINX container**:
   Once the container is created, go to the **Overview** of the ACI, and you will see the **IP address** for your container instance.

2. **Test the NGINX website**:
   Open a web browser and navigate to the IP address of the ACI. You should see the default NGINX welcome page, confirming that your container is running successfully.

---

### **Summary of Tasks**:

1. **Created a Virtual Machine (VM)** in Azure and installed Docker on it.
2. **Downloaded the NGINX Docker image** on the VM.
3. **Created an Azure Container Registry (ACR)** and pushed the NGINX Docker image to the ACR.
4. **Created an Azure Container Instance (ACI)** from the image stored in ACR and accessed the running NGINX container.

By following these steps, you've successfully completed the assignment. Let me know if you need further assistance or clarification on any step!
