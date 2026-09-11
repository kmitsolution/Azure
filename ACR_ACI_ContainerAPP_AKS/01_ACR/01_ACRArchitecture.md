# Azure Container Registry (ACR) — Architecture Overview

Azure Container Registry (ACR) is Azure's **private container image registry** used to store, manage, secure, and distribute container images and other OCI artifacts.

Think of ACR as the **private Docker Hub for your Azure environment**, but tightly integrated with Azure services such as AKS, Container Apps, App Service, Container Instances, and DevOps/GitHub pipelines.

## 1. High-Level ACR Architecture

A typical production architecture looks like this:

```text
                 Developers
                     |
                     | docker build
                     v
              +---------------+
              | Docker / Build|
              +-------+-------+
                      |
                      | docker push
                      v
          +-------------------------+
          | Azure Container Registry|
          |          (ACR)          |
          +-----------+-------------+
                      |
              Stores container
                 images
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
     AKS       Container Apps      App Service
       |              |              |
       +--------------+--------------+
                      |
                      v
                  Containers
```

The basic workflow is:

```text
Source Code
    ↓
Docker Build
    ↓
Container Image
    ↓
ACR
    ↓
Container Platform
    ↓
Running Container
```

---

# 2. What Does ACR Actually Store?

ACR primarily stores **container images and OCI artifacts**.

For example:

```text
myapp:v1
myapp:v2
backend:1.0
frontend:2026-09-10
nginx:latest
```

Internally, an image consists of multiple layers.

For example:

```text
              myapp:v1
                  |
        +---------+---------+
        |                   |
      Manifest          Image Config
        |
   +----+----+----+
   |    |    |    |
   v    v    v    v
 Layer Layer Layer Layer
```

Docker uses these layers so that unchanged layers don't have to be uploaded repeatedly.

For example:

```text
Image v1

Layer A
Layer B
Layer C
```

Then you release v2:

```text
Image v2

Layer A  ← reused
Layer B  ← reused
Layer C  ← reused
Layer D  ← new
```

This makes image distribution more efficient.

---

# 3. ACR Repository Structure

An ACR registry can contain multiple repositories.

For example:

```text
mycompany.azurecr.io
│
├── frontend
│   ├── v1
│   ├── v2
│   └── v3
│
├── backend
│   ├── v1
│   └── v2
│
├── payment-service
│   ├── 1.0
│   └── 1.1
│
└── nginx
    └── latest
```

The fully qualified image name could be:

```text
mycompany.azurecr.io/backend:v2
```

Here:

```text
mycompany.azurecr.io
        ↑
      ACR Login Server

backend
        ↑
    Repository

v2
        ↑
      Tag
```

---

# 4. ACR and Azure Virtual Network

By default, ACR is an Azure-managed service.

Your ACR does **not need to be inside your VNet** just because your application is running inside a VNet.

For example:

```text
Azure VNet
+--------------------------------------+
|                                      |
|       AKS                             |
|       +-------------+                |
|       | Worker Nodes|                |
|       +------+------+                |
|              |                       |
+--------------|-----------------------+
               |
               | HTTPS
               |
               v
       +------------------+
       |       ACR        |
       | Azure Managed   |
       |     Service     |
       +------------------+
```

However, for more secure production environments, you can use an **ACR Private Endpoint**.

Then the architecture becomes:

```text
                  Azure VNet
+------------------------------------------------+
|                                                |
|     AKS                                        |
|      |                                         |
|      | Pull Image                              |
|      v                                         |
|   Private Endpoint                             |
|      |                                         |
|      +--------------------+                    |
|                           |                    |
+---------------------------|--------------------+
                            |
                            v
                    +---------------+
                    |      ACR      |
                    | Private Access|
                    +---------------+
```

This allows registry access through a private IP rather than exposing registry traffic publicly.

---

# 5. ACR + AKS Architecture

One of the most common real-world scenarios is:

```text
                 Developer
                     |
                     v
              Git Repository
                     |
                     v
             CI/CD Pipeline
                     |
                Docker Build
                     |
                     v
                    ACR
                     |
             docker image
                     |
                     v
                    AKS
                     |
              +------+------+
              |             |
           Pod 1          Pod 2
              |             |
              +------+------+
                     |
                 Application
```

For example:

```text
Dockerfile
    |
    | docker build
    v
myapp:v10
    |
    | docker push
    v
mycompany.azurecr.io/myapp:v10
    |
    | AKS pulls image
    v
+----------------------+
| Kubernetes Pod       |
|                      |
| myapp:v10            |
+----------------------+
```

### Important point

ACR **doesn't run your container**.

ACR stores the image.

AKS, Container Apps, App Service, or another compute service runs the container.

---

# 6. Authentication Between AKS and ACR

You don't normally want developers manually providing passwords to AKS.

Azure provides identity-based access.

A common architecture is:

```text
             AKS
              |
              | Managed Identity
              |
              v
       Microsoft Entra ID
              |
              | Authorization
              v
             ACR
```

The AKS identity can be granted the appropriate permission to pull images from ACR.

Conceptually:

```text
AKS Identity
     |
     | AcrPull
     v
ACR
```

This is much better than putting registry credentials inside Kubernetes manifests.

---

# 7. ACR + CI/CD

A production CI/CD architecture could look like:

```text
Developer
    |
    v
GitHub / Azure Repos
    |
    v
CI/CD Pipeline
    |
    +---- Build
    |
    +---- Test
    |
    +---- Security Scan
    |
    +---- Docker Image
             |
             v
       +-----------+
       |    ACR    |
       +-----+-----+
             |
             v
       Deployment
             |
             v
            AKS
```

For example:

```text
git push
   ↓
Pipeline triggered
   ↓
docker build
   ↓
docker test
   ↓
docker push
   ↓
ACR
   ↓
AKS deployment
```

---

# 8. ACR Geo-Replication

For applications deployed across multiple Azure regions, ACR supports **geo-replication** with appropriate registry configuration/tier.

Example:

```text
                    ACR
                     |
          +----------+----------+
          |                     |
          v                     v
      East US                West US
     Registry               Registry
          |                     |
          v                     v
        AKS                   AKS
     Cluster 1              Cluster 2
```

This is useful when your application is deployed globally.

Instead of having your West US AKS cluster repeatedly pull images from a distant region, images can be available from a geographically closer registry replica.

Example:

```text
Users
  |
  +-------------------+
  |                   |
  v                   v
AKS East US        AKS West US
  |                   |
  v                   v
ACR East US        ACR West US
```

---

# 9. ACR Security Architecture

A production ACR environment can include several layers of security:

```text
                 Developer
                     |
                     v
              Microsoft Entra ID
                     |
                     v
               RBAC / Identity
                     |
                     v
              +-------------+
              |     ACR     |
              +-------------+
               /     |      \
              /      |       \
             v       v        v
       Private    Image     Content
       Endpoint   Scanning  Trust/Security
```

Important security concepts include:

### Microsoft Entra ID

Used for identity and authentication.

### Azure RBAC

Controls who can perform operations against the registry.

Examples include permissions for:

```text
Pull images
Push images
Manage registry
```

### Private Endpoint

Provides private network connectivity to the registry.

### Firewall / Network Access Controls

Can restrict how the registry is accessed depending on the configuration.

### Image Security

You should also consider:

```text
Image
  ↓
Vulnerability scanning
  ↓
Approved image
  ↓
Production deployment
```

---

# 10. ACR Tasks

ACR can also provide build capabilities through **ACR Tasks**.

Instead of always building an image locally:

```text
Developer
    |
    v
Source Code
    |
    v
ACR Tasks
    |
    v
Container Image
    |
    v
ACR
```

This is useful for automated container builds.

For example:

```text
Git commit
    ↓
ACR Task
    ↓
Build Docker image
    ↓
Push image to ACR
```

---

# 11. ACR vs Docker Hub

A useful way to explain ACR to students is:

| Feature                     | Docker Hub                       | Azure Container Registry |
| --------------------------- | -------------------------------- | ------------------------ |
| Container registry          | ✅                                | ✅                        |
| Private repositories        | ✅                                | ✅                        |
| Azure integration           | Limited                          | Excellent                |
| Azure RBAC                  | ❌                                | ✅                        |
| Managed Identity            | ❌                                | ✅                        |
| Private Endpoint            | ❌                                | ✅                        |
| Geo-replication             | Available in some plans/features | ✅                        |
| AKS integration             | Possible                         | Native Azure integration |
| Enterprise Azure networking | Limited                          | Strong                   |

So in an Azure enterprise environment:

```text
Azure Workload
      |
      v
     ACR
      |
      v
Azure Compute
```

is a very common architecture.

---

# 12. Complete Production Architecture

A more realistic enterprise architecture could be:

```text
                         Developers
                             |
                             v
                      GitHub / Azure DevOps
                             |
                             v
                        CI/CD Pipeline
                             |
                    +--------+--------+
                    |                 |
                  Build             Test
                    |                 |
                    +--------+--------+
                             |
                             v
                    +------------------+
                    |       ACR        |
                    |                  |
                    | Container Images |
                    | OCI Artifacts    |
                    +--------+---------+
                             |
                    Private Endpoint
                             |
                    +--------+---------+
                    |      VNet        |
                    |                  |
                    |      AKS         |
                    |   +----------+   |
                    |   | Pod      |   |
                    |   | Pod      |   |
                    |   | Pod      |   |
                    |   +----------+   |
                    |                  |
                    +------------------+
                             |
                             v
                           Users
```

---

# 13. The Most Important Concept

For your Azure course, I would emphasize this distinction:

```text
             ACR
              |
       "Where is my
        container image?"
              |
              v
       Container Image
              |
              |
              v
       +--------------+
       | Compute      |
       +--------------+
        /     |      \
       /      |       \
     AKS   Container  App Service
           Apps
```

**ACR = Store and distribute container images**

**AKS / Container Apps / App Service / ACI = Run containers**

That distinction is fundamental.

---

# 14. ACR Architecture Learning Path

For an Azure administrator/DevOps course, I recommend teaching ACR in this order:

### Module 1 — Fundamentals

1. What is Azure Container Registry?
2. ACR vs Docker Hub
3. Registry → Repository → Image → Tag
4. ACR SKUs/Tiers
5. Creating an ACR

### Module 2 — Container Images

6. Docker build
7. Docker login
8. Docker push
9. Docker pull
10. Image tags and versioning
11. Image layers

### Module 3 — Security

12. Microsoft Entra authentication
13. Azure RBAC
14. Managed Identity
15. ACR permissions
16. Private Endpoint
17. Network restrictions
18. Image vulnerability/security scanning

### Module 4 — ACR + Azure Compute

19. ACR + Azure VM
20. ACR + ACI
21. ACR + App Service
22. **ACR + AKS**
23. ACR + Azure Container Apps

### Module 5 — Advanced

24. ACR Tasks
25. Geo-replication
26. Webhooks
27. Image retention
28. Importing images
29. CI/CD integration
30. Production architecture

For your **AZ-104/Azure administration teaching**, I would make the next lecture **“ACR Hands-on: Create Registry → Create Docker Image → Push Image → Pull Image → Run Container”**, because it connects the architecture directly to a practical lab.
