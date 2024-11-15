### **Assignment: Working with Azure DevOps, GitHub, Azure Repos, and Docker Integration**

#### **Objective:**
The objective of this assignment is to familiarize yourself with creating a .NET MVC application in Visual Studio, managing the source code using GitHub and Azure Repos, setting up an Azure DevOps pipeline, containerizing the application using Docker, and deploying the container to Azure Kubernetes Service (AKS) using Azure Container Registry (ACR).

---

### **Part 1: Create MVC Application in .NET using Visual Studio**

1. Create a new ASP.NET Core MVC application in Visual Studio.
2. Verify that the application runs locally and displays the default MVC page.

---

### **Part 2: Push Code to GitHub Repository**

1. Create a new repository on GitHub.
2. Push your local .NET MVC application to this GitHub repository from Visual Studio.

---

### **Part 3: Import GitHub Repository to Azure Repos**

1. Create a new Azure DevOps project and repository.
2. Import your GitHub repository into Azure Repos.
3. Verify that the code has been successfully imported into Azure Repos.

---

### **Part 4: Update References to Azure Repo and Change Default Branch to `main`**

1. Remove the reference to the GitHub repository from your local repository.
2. Connect your local repository to the newly created Azure Repo.
3. Change the default branch of your repository to `main` both locally and in Azure Repos.
4. Push the code to Azure Repos and verify that the `main` branch is created.

---

### **Part 5: Create and Run Azure Pipeline**

1. Create a new Azure DevOps pipeline to build your application using the Microsoft-hosted agent.
2. Verify that the pipeline runs successfully.
3. Remove the pipeline YAML file from your Azure Repo.

---

### **Part 6: Create Docker Image and Push to Azure Container Registry (ACR)**

1. Create a Dockerfile to containerize your .NET MVC application.
2. Build the Docker image locally and ensure it runs.
3. Push the Docker image to Azure Container Registry (ACR).

---

### **Part 7: Deploy the Docker Image Using Kubernetes on Azure**

1. Create an Azure Kubernetes Service (AKS) cluster.
2. Deploy the Docker image stored in ACR to AKS.
3. Configure a LoadBalancer service for your Kubernetes deployment.
4. Verify that the application is accessible via the LoadBalancer's external IP address.

--- 

### **Submission Instructions:**
- Submit a detailed report of the steps followed.
- Include the relevant code snippets (Dockerfile, YAML pipeline, etc.).
- Provide screenshots or logs where applicable, including successful build and deployment outputs.
