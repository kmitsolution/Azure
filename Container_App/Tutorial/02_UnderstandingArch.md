# Understanding Azure Container Apps Architecture

Azure Container Apps (ACA) is built on top of Kubernetes, but Microsoft hides the Kubernetes complexity from you.

The architecture looks like this:

![Image](https://images.openai.com/static-rsc-4/ByNnlIf1U2HtEmW0zw_59F_H7Yq4HI2tt3-OpkjAIQ3V5Bri2YX3C495xdB5IpvQmTUpX9FwxHwOBhZH8CjJgQtsoTUmawfhiVimWerxDsK9yY6r11pYRDPLGUOrIrA9jFYPXgYTrcVQ7Ln0A9GmrRn-tmdqUVpK-RHThB1rrPfzE1Z6u89uqFblsCNoRH0G?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/D8Wnv2DK-I-hF4ayOmgHMS6kilC1HNg3wzQ4ct4FtnKbt2FpxodSaHD3ziQxQsAL-ikeZEFMVEnd9t9jNFzqZJECgqd5U_f0bpGI39tPoDLOYR7CELQhnBLoqNezg-YSEaZtGoRFClGqNbzzAlOxfWXvT8wULyT0Dw3fCJBVlwhzxvtcuyCM9Mbl-NCVH8Lx?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/XflMwt0zgaUTLvOXCCjpv3Nu0_ZVuYvwflGvpsTXyh6rlqlzKgsG7fnK8WS3Q8MNJ1-ucBOxwr4N-zx3Hz7kOJ0Wg8WkiHUNnmDxjBop2KN83nzeU1ZdWHczLPMmUNb3iECIhRu-zwmWZB3o9l1B2hs9xaE5Uhcc8tclHeC_vb2jTPm4qHVKw9cfxVnvzCib?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/sCWe4UTAIug3vUJR6B8Q6nC8UIpzBGyl0OsyI_W92m0YVF8T6QrT-oBge_pwY_LQrxzoutKmtXeeGxTht9Xw_S3AZONCr5JAW_-Uss4N1VCVeKS4FLvmiPprsz93G8vcNpyQ9oUTaSXdwW3yD-SsQ9gKfFEgItWcfTy-r9xejkpGf6XJgdKx6t1eSdfE3lp5?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ZkZTyf52FeMyKDhAoBtEpGy_XIQoIqjJaxxw9t4LhS2cUQMdTeHlc8xLIP5Pot-38L9sD3azZzgfxdRFantQmNjwieVEQbOmSTk4FHGm7FR335vuSY2kIaTPY8duA9YVTdrRwuTr0ml8UsBg6c000z-CZlL1g7Ikg8r_WY1jmNqcVb6RGExru5iNn8VW-4B0?purpose=fullsize)

```text
Internet Users
       │
       ▼
Ingress (HTTPS)
       │
       ▼
Azure Container App
       │
       ▼
Container App Environment
       │
       ▼
Managed Kubernetes Infrastructure
       │
       ▼
Azure Infrastructure
```

---

# High-Level Architecture

```text
+----------------------------------------------------+
|                 Azure Subscription                 |
|                                                    |
|  Resource Group                                    |
|       │                                            |
|       ├── Container App Environment                |
|       │         │                                  |
|       │         ├── Container App 1               |
|       │         ├── Container App 2               |
|       │         ├── Container App 3               |
|       │         └── Log Analytics Workspace       |
|       │                                            |
|       ├── Azure Container Registry                 |
|       ├── Key Vault                                |
|       ├── Storage Account                          |
|       └── Service Bus                              |
+----------------------------------------------------+
```

---

# Main Components

## 1. Container App Environment (Very Important)

Think of this like a **logical boundary** or **cluster**.

It is similar to:

```text
AKS Cluster
```

but Microsoft manages it.

Inside one environment, you can deploy multiple container apps.

Example:

```text
Container App Environment
       │
       ├── customer-service
       ├── payment-service
       ├── notification-service
       └── frontend-service
```

### Responsibilities

* Networking
* Logging
* Internal DNS
* Scaling Infrastructure
* Dapr Components
* VNET Integration

---

### Example:

```bash
az containerapp env create
```

This creates the underlying infrastructure.

---

# 2. Container App

A Container App is your actual application.

Equivalent to:

```text
Deployment + Service (Kubernetes)
```

Example:

```text
nginx
dotnet api
python flask
node js app
```

Container Apps run container images from:

* Docker Hub
* Azure Container Registry
* Private Registry

Example:

```text
myacr.azurecr.io/payment:v1
```

---

# 3. Revisions

Every deployment creates a new revision.

Similar to:

```text
Kubernetes ReplicaSet
```

Example:

```text
Revision 1
Image: v1

Revision 2
Image: v2
```

Benefits:

* Rollback
* Canary deployments
* Blue-Green deployments

---

Example:

```text
v1 → 80%
v2 → 20%
```

---

# 4. Replicas

A revision may have multiple replicas.

Example:

```text
payment-service
      │
      ├── Replica 1
      ├── Replica 2
      └── Replica 3
```

These replicas scale automatically.

---

# 5. Ingress

Ingress exposes your application.

Two types:

---

## External Ingress

```text
Internet
     │
     ▼
Container App
```

Public URL is generated.

Example:

```text
https://payment.redflower123.centralindia.azurecontainerapps.io
```

---

## Internal Ingress

```text
Frontend
      │
      ▼
Payment Service
```

Accessible only within the environment.

---

# 6. Environment Internal DNS

Every app gets an internal DNS name.

Example:

```text
payment-service
customer-service
notification-service
```

Microservices can communicate directly.

Example:

```text
http://payment-service
```

No Kubernetes service creation needed.

---

# 7. KEDA Autoscaler

Azure Container Apps uses:

## KEDA

(Kubernetes Event Driven Autoscaler)

It automatically scales based on:

* HTTP Requests
* Service Bus Messages
* Kafka Messages
* RabbitMQ Queue Length
* Azure Queue
* Cron Jobs

Architecture:

```text
Queue
   │
   ▼
KEDA
   │
   ▼
Scale Replicas
```

---

# 8. Dapr Sidecar (Optional)

Dapr provides:

* Service Discovery
* Pub/Sub
* State Management
* Distributed Tracing
* Secret Store

Architecture:

```text
Application Container
          │
          ▼
      Dapr Sidecar
```

Similar to:

```text
Istio Sidecar
```

---

# 9. Log Analytics Workspace

All logs are stored here.

```text
Application Logs
System Logs
Scaling Logs
Console Logs
```

Architecture:

```text
Container App
      │
      ▼
Log Analytics
      │
      ▼
Azure Monitor
```

---

# 10. Managed Environment Infrastructure

Microsoft internally creates:

```text
Kubernetes Cluster
Nodes
Ingress Controller
Control Plane
```

You cannot directly access them.

This is hidden.

---

# Actual Internal Architecture

```text
Container Apps
       │
       ▼
Managed Kubernetes Cluster
       │
       ▼
KEDA
       │
       ▼
Envoy Proxy
       │
       ▼
Dapr
       │
       ▼
Azure Infrastructure
```

---

# End-to-End Architecture Example

```text
Internet User
      │
      ▼
Azure Front Door
      │
      ▼
Container App Frontend
      │
      ▼
Payment Service
      │
      ▼
Service Bus
      │
      ▼
Notification Service
      │
      ▼
Storage Account
```

---

# Banking Example

```text
Container App Environment
│
├── frontend-api
├── customer-api
├── account-api
├── transaction-api
├── notification-api
└── worker-service
```

Worker service:

```text
Service Bus Message
        │
        ▼
Scale Out
        │
        ▼
Process Message
        │
        ▼
Scale to Zero
```

This is one of the biggest advantages of Container Apps.

---

# Mapping with Kubernetes

| Azure Container Apps | Kubernetes Equivalent |
| -------------------- | --------------------- |
| Environment          | Cluster/Namespace     |
| Container App        | Deployment + Service  |
| Revision             | ReplicaSet            |
| Replica              | Pod                   |
| Ingress              | Ingress Controller    |
| KEDA Scaling         | HPA/KEDA              |
| Jobs                 | CronJobs              |
| Dapr                 | Service Mesh Features |

---

# Architecture Diagram (Complete)

```text
                Internet
                     │
                     ▼
            Azure Front Door
                     │
                     ▼
             External Ingress
                     │
 ┌───────────────────────────────────┐
 │ Container App Environment         │
 │                                   │
 │   ┌──────────────┐                │
 │   │ Frontend API │                │
 │   └──────────────┘                │
 │            │                      │
 │            ▼                      │
 │   ┌──────────────┐                │
 │   │ Payment API  │                │
 │   └──────────────┘                │
 │            │                      │
 │            ▼                      │
 │   ┌──────────────┐                │
 │   │ Worker App   │                │
 │   └──────────────┘                │
 │                                   │
 │ Internal DNS + KEDA + Dapr        │
 └───────────────────────────────────┘
                     │
      ┌──────────────┼──────────────┐
      ▼              ▼              ▼
 Service Bus      Key Vault      Storage
```

---

# Important Interview Question

**Q: Is Azure Container Apps using Kubernetes internally?**

**Answer:**

Yes. Azure Container Apps internally runs on managed Kubernetes infrastructure and uses:

* Kubernetes
* KEDA
* Envoy
* Dapr

However, Microsoft abstracts all Kubernetes management activities such as:

* Node management
* Upgrades
* Control plane
* Networking configuration
* Ingress controllers

So developers can run containers without directly managing Kubernetes.
