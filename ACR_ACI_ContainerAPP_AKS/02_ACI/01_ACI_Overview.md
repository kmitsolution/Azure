# Azure Container Instances (ACI) — Overview

**Azure Container Instances (ACI)** is an Azure service that lets you **run containers directly in Azure without managing virtual machines or a Kubernetes cluster**.

The simplest way to remember it:

> **ACR stores the container image. ACI runs the container.**

```text
                 Container Image
                       |
                       v
              +------------------+
              |       ACR        |
              | myapp:v1         |
              +--------+---------+
                       |
                       | Pull
                       v
              +------------------+
              |       ACI        |
              |                  |
              |    Container     |
              |     myapp        |
              +--------+---------+
                       |
                       v
                    Users
```

---

# 1. Why Do We Need ACI?

Suppose you have created a Docker image:

```text
myapp:v1
```

You want to run it in Azure.

Traditionally, you might create:

```text
VM
 ↓
Install Docker
 ↓
Configure Docker
 ↓
Pull Image
 ↓
Run Container
```

That's a lot of infrastructure to manage.

With ACI:

```text
Docker Image
     ↓
    ACI
     ↓
 Running Container
```

Azure manages the underlying infrastructure for you.

---

# 2. ACI vs Virtual Machine

This is an important comparison.

### VM approach

```text
                Azure VM
        +-----------------------+
        | Operating System      |
        | Docker                |
        | Container             |
        +-----------------------+
```

You are responsible for things such as:

* VM configuration
* OS maintenance
* Docker installation/configuration
* Patching
* VM availability

### ACI approach

```text
                Azure
        +-----------------------+
        | Managed Infrastructure|
        |                       |
        |       Container       |
        +-----------------------+
```

Azure handles the underlying infrastructure.

You primarily focus on:

```text
Container
Image
CPU
Memory
Networking
Environment variables
```

---

# 3. ACI vs AKS

This is probably the **most important comparison** for your Azure course.

| Feature                             | ACI                | AKS                                                  |
| ----------------------------------- | ------------------ | ---------------------------------------------------- |
| Run containers                      | ✅                  | ✅                                                    |
| Kubernetes                          | ❌                  | ✅                                                    |
| VM management                       | Managed            | Azure manages nodes/control plane, depending on mode |
| Complexity                          | Low                | Higher                                               |
| Startup                             | Very fast          | More infrastructure                                  |
| Scaling                             | Limited/simple     | Advanced                                             |
| Load balancing                      | Basic capabilities | Advanced                                             |
| Service discovery                   | Limited            | Kubernetes                                           |
| Suitable for microservices platform | Limited            | ✅                                                    |
| Short-lived workloads               | ✅                  | ✅                                                    |
| Large production container platform | Usually not ideal  | ✅                                                    |

Think:

```text
Simple container
      ↓
     ACI
```

versus:

```text
Many containers
      ↓
Microservices
      ↓
Kubernetes
      ↓
     AKS
```

---

# 4. ACI Architecture

A simple ACI architecture:

```text
                    Azure
                      |
              +-------+-------+
              |       |       |
              v       v       v
          Container Container Container
             1         2         3
              \         |        /
               \        |       /
                +-------+------+
                        |
                  Container Group
```

One important ACI concept is the **container group**.

---

# 5. What is a Container Group?

A **container group** is a group of containers that share certain resources and lifecycle characteristics.

For example:

```text
             Container Group
                    |
          +---------+---------+
          |                   |
          v                   v
       Nginx              Application
       Container           Container
```

Containers in the same group can share:

* Network namespace
* IP address
* Port namespace
* Volumes

This makes a container group somewhat similar to the idea of a **pod** in Kubernetes.

A useful mental model is:

```text
ACI Container Group
        ≈
Kubernetes Pod
```

They aren't identical, but the analogy is useful for learning.

---

# 6. Example: Web + Sidecar

Suppose you want:

```text
             ACI Container Group
                     |
             +-------+-------+
             |               |
             v               v
          Web App          Logging
          Container        Container
```

Both containers are part of the same group.

They can communicate over the shared network environment.

---

# 7. ACI with ACR

This is where ACR and ACI fit together perfectly.

We previously learned:

```text
ACR
 ↓
Container Image
```

Now:

```text
ACR
 |
 | myapp:v1
 |
 | Pull
 v
ACI
 |
 v
Container
```

Complete architecture:

```text
                 Developer
                     |
                     v
                Docker Build
                     |
                     v
             myreg07.azurecr.io
                     |
                     v
                    ACR
                     |
                  myapp:v1
                     |
                     | Pull
                     v
                    ACI
                     |
             +-------+-------+
             |               |
             v               v
          Container       Container
             |
             v
           Users
```

---

# 8. Private ACR + ACI

For a production-style architecture, you don't necessarily want a public container registry.

You can use:

```text
                 ACR
                  |
            Private Access
                  |
                  v
                VNet
                  |
                  v
                 ACI
```

The exact networking design depends on how the ACI deployment is configured.

This gives you:

```text
Private Container Image
        +
Private/controlled networking
        +
Managed container execution
```

---

# 9. ACI Networking

ACI supports networking options including public networking and VNet integration scenarios.

A basic deployment might look like:

```text
Internet
   |
   | HTTP
   v
+---------+
|   ACI   |
|         |
| Port 80 |
+---------+
```

The container can receive a public IP depending on the configuration.

For example:

```text
Public IP
   |
   v
ACI Container
   |
   v
Web Application
```

For internal applications:

```text
Azure VNet
    |
    v
ACI
    |
    v
Private Application
```

---

# 10. ACI CPU and Memory

When creating an ACI container, you specify resources such as:

```text
CPU
Memory
```

For example:

```text
CPU:    1
Memory: 1 GB
```

Conceptually:

```text
ACI Container
+--------------------+
| CPU: 1             |
| Memory: 1 GB       |
|                    |
| Application        |
+--------------------+
```

This is different from creating a VM where you select a VM size such as:

```text
Standard_B2s
Standard_D2s_v5
```

With ACI, you are thinking more directly in terms of **container resources**.

---

# 11. ACI is Serverless Containers

ACI is often described as a **serverless container** service.

That doesn't mean there are literally no servers.

It means:

> **You don't have to provision or manage the underlying servers.**

Conceptually:

```text
You
 |
 +---- Container Image
 +---- CPU
 +---- Memory
 +---- Network
 |
 v
Azure
 |
 +---- Infrastructure
 +---- Hosts
 +---- OS
 +---- Container runtime
```

Azure handles the underlying infrastructure.

---

# 12. When Should You Use ACI?

ACI is excellent for:

### Quick testing

```text
Developer
   ↓
Container Image
   ↓
ACI
   ↓
Test
```

No need to create a VM.

---

### Short-lived workloads

For example:

```text
Job
 ↓
Run Container
 ↓
Complete
 ↓
Remove Container
```

This is a good fit for certain batch-style or temporary workloads.

---

### CI/CD jobs

You can use containers for temporary build/test workloads in suitable architectures.

```text
Pipeline
   |
   v
ACI
   |
   +---- Run tests
   |
   +---- Finish
   |
   v
Delete
```

---

### Simple applications

If you have a single containerized application and don't need Kubernetes:

```text
Internet
   |
   v
ACI
   |
Container
   |
Application
```

ACI can be much simpler than deploying AKS.

---

# 13. When Should You NOT Use ACI?

Don't automatically choose ACI for every container workload.

If you need:

```text
Many microservices
Complex orchestration
Advanced autoscaling
Rolling deployments
Service discovery
Complex traffic management
Kubernetes ecosystem
```

then **AKS** is usually a better fit.

Think:

```text
1 simple container
       ↓
      ACI
```

versus:

```text
100+ containers
       ↓
Microservices
       ↓
     AKS
```

---

# 14. ACI Lifecycle

The lifecycle is very simple:

```text
Create
  ↓
Starting
  ↓
Running
  ↓
Stopped
  ↓
Deleted
```

For example:

```text
ACI Container
     |
     | Start
     v
   Running
     |
     | Work complete
     v
   Stopped
     |
     | Delete
     v
   Removed
```

---

# 15. ACI Billing Concept

ACI is generally billed based on the resources consumed by the container workload, rather than requiring you to maintain a VM continuously.

Think:

```text
Container
   |
   +---- CPU usage
   +---- Memory
   +---- Runtime
```

This can be attractive for short-lived workloads because you don't need to keep a VM running just to host the container.

Always check the current Azure pricing for the exact region and configuration before making cost decisions.

---

# 16. ACI + Managed Identity

Just like our previous VM lab, ACI can use **managed identities** in supported scenarios.

The architecture can become:

```text
                ACI
                 |
                 | Managed Identity
                 v
          Microsoft Entra ID
                 |
          +------+------+
          |             |
          v             v
         ACR        Key Vault
```

For example, an ACI workload can use an identity to authenticate to Azure resources rather than embedding credentials inside the container.

This follows the same principle we discussed earlier:

```text
Identity = WHO AM I?

RBAC = WHAT CAN I ACCESS?
```

---

# 17. ACI + ACR + Managed Identity

Now we can combine everything you've learned:

```text
                         Microsoft Entra ID
                                |
                                |
                         Managed Identity
                                |
                                v
                         +-------------+
                         |     ACI     |
                         |             |
                         |  Container  |
                         +------+------+
                                |
                                | Pull
                                v
                         +-------------+
                         |     ACR     |
                         |             |
                         | myapp:v1    |
                         +-------------+
```

The flow:

```text
1. Build image
       ↓
2. Push image to ACR
       ↓
3. Create ACI
       ↓
4. Give ACI identity permission
       ↓
5. ACI authenticates
       ↓
6. ACI pulls image
       ↓
7. Container starts
```

---

# 18. ACI vs VM vs AKS

This is a great exam/interview table:

| Requirement                     |       VM |         ACI |                         AKS |
| ------------------------------- | -------: | ----------: | --------------------------: |
| Run normal VM workloads         |        ✅ |           ❌ |                           ❌ |
| Run a simple container          | Possible | ⭐ Excellent |                    Possible |
| No VM management                |        ❌ |           ✅ |                           ✅ |
| Kubernetes                      |        ❌ |           ❌ |                           ✅ |
| Simple deployment               |   Medium | ⭐ Very easy |                     Complex |
| Multiple microservices          | Possible |     Limited |                 ⭐ Excellent |
| Advanced orchestration          |        ❌ |           ❌ |                           ⭐ |
| Short-lived container           | Possible | ⭐ Excellent |                    Possible |
| Full OS control                 |        ⭐ |           ❌ | Node-level where applicable |
| Serverless container experience |        ❌ |           ✅ |                           ❌ |

---

# ⭐ The Most Important Concept

Students often confuse these three Azure services:

```text
              Container Workload
                     |
        +------------+------------+
        |            |            |
        v            v            v
       ACI          AKS          VM
        |            |            |
    Simple        Kubernetes    Docker
   Container     Orchestration  manually
```

### ACR

**Stores the image**

```text
ACR
 ↓
myapp:v1
```

### ACI

**Runs a container**

```text
ACI
 ↓
Container
```

### AKS

**Orchestrates containers**

```text
AKS
 ↓
Pods
 ↓
Services
 ↓
Deployments
 ↓
Scaling
```

### VM

**Provides a virtual machine**

```text
VM
 ↓
OS
 ↓
Docker
 ↓
Container
```

---

# 🎯 Real-Time Example

Suppose your company has a small image-processing application.

The application needs to run a container whenever a file is uploaded.

You could design:

```text
             File Upload
                  |
                  v
              Trigger
                  |
                  v
                 ACI
                  |
                  v
        Image Processing
                  |
                  v
             Job Complete
                  |
                  v
             Container Ends
```

You don't necessarily need a continuously running VM or a full Kubernetes cluster.

That's exactly the kind of scenario where **ACI's simplicity is attractive**.

---

## Recommended Next Lecture

For your course, after this overview, I'd make the next lecture:

**"Azure Container Instances Hands-on — Deploy a Container from ACR using Portal and CLI"**

The lab can build directly on your existing `myreg07` + `mynginx` setup:

```text
myreg07 (ACR)
     |
     | mynginx:latest
     ↓
Azure Container Instance
     |
     ↓
Public IP
     |
     ↓
Browser → Nginx
```

Then we can do the **more advanced version using ACI + VNet + Managed Identity + private ACR**, which will connect very nicely with everything you've just learned.
