# Azure Container Registry — Module 1: Fundamentals

## Lecture 1 — What is Azure Container Registry?

### 1. What is ACR?

**Azure Container Registry (ACR)** is a managed, private container registry service provided by Azure.

Its primary purpose is to **store and manage container images and OCI artifacts** so that Azure services and applications can securely pull those images when they need to run containers.

Think about it like this:

```text
Docker Image
     |
     | docker push
     v
+----------------------+
| Azure Container      |
| Registry (ACR)       |
|                      |
|  frontend:v1         |
|  backend:v1          |
|  payment:v2          |
+----------+-----------+
           |
           | Pull Image
           v
      Azure Services
       /     |      \
      /      |       \
    AKS     ACI    App Service
```

### Simple definition

> **ACR is Azure's managed private registry for storing, managing, and distributing container images.**

---

# 2. Why Do We Need ACR?

Suppose you have developed a Java application.

You create a Docker image:

```text
myapp:v1
```

Your application needs to run on AKS.

Where does AKS get the image from?

You need a **container registry**.

```text
Developer
    |
    | Build
    v
Docker Image
    |
    | Push
    v
ACR
    |
    | Pull
    v
AKS
    |
    v
Container
```

Without a registry, you would need to manually copy images to every machine running your containers.

With ACR:

```text
              ACR
               |
       +-------+-------+
       |       |       |
       v       v       v
      AKS     ACI    App Service
```

The same image can be consumed by multiple Azure services.

---

# 3. What Can ACR Store?

ACR can store:

### Container Images

For example:

```text
nginx
ubuntu
myapp
backend
frontend
```

### Image Versions

```text
myapp:v1
myapp:v2
myapp:v3
```

### OCI Artifacts

ACR also supports the OCI ecosystem beyond traditional Docker images.

So conceptually:

```text
                    ACR
                     |
        +------------+------------+
        |            |            |
   Container      OCI          Other
    Images      Artifacts      Artifacts
```

---

# Lecture 2 — ACR vs Docker Hub

This is a very common interview question.

## Docker Hub

Docker Hub is a public/private container registry operated by Docker.

## Azure Container Registry

ACR is Microsoft's managed registry service integrated with Azure.

### Comparison

| Feature                      | Docker Hub                  | Azure Container Registry |
| ---------------------------- | --------------------------- | ------------------------ |
| Container registry           | ✅                           | ✅                        |
| Public repositories          | ✅                           | ✅                        |
| Private repositories         | ✅                           | ✅                        |
| Azure integration            | Limited                     | Excellent                |
| Azure RBAC                   | ❌                           | ✅                        |
| Microsoft Entra ID           | ❌                           | ✅                        |
| Managed Identity             | ❌                           | ✅                        |
| Private Endpoint             | ❌                           | ✅                        |
| Azure networking integration | Limited                     | Strong                   |
| Geo-replication              | Available depending on plan | ✅                        |
| ACR Tasks                    | ❌                           | ✅                        |
| AKS integration              | Possible                    | Native                   |

### When would you use Docker Hub?

For example:

```text
Developer
    |
    v
Docker Hub
    |
    v
Kubernetes
```

Docker Hub is particularly common for **public/open-source images**.

For example:

```text
nginx
redis
ubuntu
node
```

### When would you use ACR?

For an Azure enterprise application:

```text
Developer
     |
     v
Azure DevOps / GitHub
     |
     v
     ACR
     |
     v
     AKS
```

ACR provides tighter integration with Azure identity, networking, and compute services.

---

# Lecture 3 — Registry → Repository → Image → Tag

This is **very important**.

Many beginners confuse these four terms.

Let's understand them from the outside in.

---

## 1. Registry

The **registry** is the overall container image storage service.

Example:

```text
mycompany.azurecr.io
```

This is the ACR login server.

Think:

```text
ACR Registry
     |
     +-------------------+
     |                   |
     v                   v
 Repository          Repository
```

---

# 2. Repository

A repository is a logical collection of related images.

For example:

```text
mycompany.azurecr.io/frontend
mycompany.azurecr.io/backend
mycompany.azurecr.io/payment
```

So:

```text
mycompany.azurecr.io
        |
        +---- frontend
        |
        +---- backend
        |
        +---- payment
```

---

# 3. Image

An image is the actual container image.

For example:

```text
backend:v1
```

or:

```text
backend:v2
```

The image contains everything required to create a container:

```text
Application
+
Runtime
+
Libraries
+
Configuration
+
Dependencies
```

---

# 4. Tag

A tag identifies a particular version/reference of an image.

For example:

```text
backend:v1
backend:v2
backend:v3
```

Here:

```text
backend
   ↑
Repository

v1
 ↑
Tag
```

---

# Complete Example

Consider:

```text
mycompany.azurecr.io/backend:v2
```

Break it down:

```text
mycompany.azurecr.io
        |
        | Registry
        |
        +---- backend
                |
                | Repository
                |
                +---- v2
                     |
                     Tag
```

So the hierarchy is:

```text
Registry
   ↓
Repository
   ↓
Image
   ↓
Tag
```

A slightly more technically precise way to teach this is:

> **Registry contains repositories; repositories contain image manifests referenced by tags/digests.**

---

# Tag vs Digest

There is another important concept.

You can reference an image using a tag:

```text
mycompany.azurecr.io/backend:v2
```

But tags can be changed.

A more immutable reference is an image **digest**:

```text
mycompany.azurecr.io/backend@sha256:abc123...
```

Conceptually:

```text
Tag
 ↓
backend:v2
 ↓
Digest
 ↓
sha256:abc123...
```

For production deployments, immutable digests are useful when you need to guarantee exactly which image was deployed.

---

# Lecture 4 — ACR SKUs / Tiers

ACR offers different service tiers.

The main tiers are:

```text
Basic
Standard
Premium
```

Think about them as increasing levels of capability.

```text
             Premium
            /       \
       Standard     More advanced
          /
       Basic
```

## Basic

Designed for:

* Development
* Testing
* Learning
* Small workloads

You get the core registry functionality but with lower capacity/performance.

---

## Standard

Designed for:

* Production workloads
* Larger image storage requirements
* Higher throughput requirements

It provides more capacity and performance than Basic.

---

## Premium

Designed for:

* Enterprise workloads
* High-performance scenarios
* Advanced networking/security requirements
* Geo-replication
* Large-scale environments

Premium is particularly important when you need advanced capabilities such as **private endpoints and geo-replication**.

---

# Basic vs Standard vs Premium

A simplified teaching table:

| Capability           |   Basic | Standard | Premium |
| -------------------- | ------: | -------: | ------: |
| Container images     |       ✅ |        ✅ |       ✅ |
| Private registry     |       ✅ |        ✅ |       ✅ |
| Production use       | Limited |        ✅ |       ✅ |
| Higher throughput    |         |        ✅ |       ✅ |
| Geo-replication      |       ❌ |        ❌ |       ✅ |
| Private Endpoint     |       ❌ |        ❌ |       ✅ |
| Enterprise scenarios |         |          |       ✅ |

**Important:** Azure can change feature availability and SKU capabilities over time, so when teaching current Azure labs, verify the exact feature/SKU matrix in the Azure documentation.

---

# Lecture 5 — Creating an ACR

Now let's create our first registry.

## Option 1 — Azure Portal

Go to:

**Azure Portal → Create a resource → Container Registry**

You will see:

```text
Create Container Registry
```

---

## Step 1 — Basics

Select:

### Subscription

Choose your Azure subscription.

Example:

```text
Azure Subscription
```

### Resource Group

Create or select:

```text
rg-acr-demo
```

### Registry Name

Enter a globally unique name.

For example:

```text
ramanacr2026
```

The name must satisfy Azure's naming requirements.

Your registry login server will look like:

```text
ramanacr2026.azurecr.io
```

### Location

Choose the Azure region.

For example:

```text
Central India
```

### SKU

For our learning lab:

```text
Basic
```

---

# Step 2 — Review + Create

Click:

```text
Review + create
```

Then:

```text
Create
```

Azure will deploy the registry.

---

# Step 3 — Open the Registry

After deployment:

```text
Azure Portal
     ↓
Container Registries
     ↓
ramanacr2026
```

You should see the ACR overview page.

One of the important properties is:

```text
Login server

ramanacr2026.azurecr.io
```

This is what you'll use when working with Docker.

---

# Step 4 — Repository

Initially your registry may not contain any repositories.

After you push your first image:

```text
ramanacr2026.azurecr.io/myapp:v1
```

ACR will have:

```text
Repositories
     |
     +---- myapp
            |
            +---- v1
```

---

# Step 5 — Important ACR Commands

After creating ACR, you'll start working with Azure CLI and Docker.

Login to Azure:

```bash
az login
```

Login to ACR:

```bash
az acr login --name ramanacr2026
```

Build an image:

```bash
docker build -t myapp:v1 .
```

Tag it for ACR:

```bash
docker tag myapp:v1 ramanacr2026.azurecr.io/myapp:v1
```

Push it:

```bash
docker push ramanacr2026.azurecr.io/myapp:v1
```

Now your architecture becomes:

```text
                 Local Machine
                      |
                Docker Build
                      |
                      v
                  myapp:v1
                      |
                 Docker Tag
                      |
                      v
          ramanacr2026.azurecr.io
                      |
                      | docker push
                      v
              +---------------+
              |      ACR      |
              |               |
              |    myapp      |
              |      |        |
              |     v1        |
              +-------+-------+
                      |
                      | docker pull
                      v
                     AKS
```

---

# ⭐ Key Concepts to Remember

For students, I would summarize Module 1 with these five points:

### 1. ACR

> **ACR is Azure's managed private container registry.**

### 2. Purpose

> **ACR stores and distributes container images; it does not run the containers.**

### 3. Structure

```text
Registry
   ↓
Repository
   ↓
Image
   ↓
Tag / Digest
```

### 4. Tiers

```text
Basic → Standard → Premium
```

Choose based on capacity, performance, and advanced requirements.

### 5. Integration

```text
             ACR
              |
      +-------+-------+
      |       |       |
     AKS     ACI    App Service
```

ACR becomes especially powerful when combined with **Microsoft Entra ID, Managed Identity, RBAC, Private Endpoint, AKS, and CI/CD**.

## Next Lecture

I recommend making **Module 1, Lecture 6 — "ACR Hands-on: Create ACR + Docker Login + Build Image + Tag Image + Push Image + Verify Repository"**. This will take the students from the architecture concepts above to their first complete container-image workflow.
