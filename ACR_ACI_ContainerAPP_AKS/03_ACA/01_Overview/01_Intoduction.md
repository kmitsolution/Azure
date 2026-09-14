 **Azure Container Apps (ACA) is a managed container application platform. It uses Kubernetes technology underneath, but you do NOT manage an AKS cluster.**

## 1. What is Azure Container Apps?

![Image](https://images.openai.com/static-rsc-4/R6gZk6_nS6z6jW1sHC5e0Uy_I_87aEsqISBQW8ydT6arm7kQw1fNasC7uDE54BeNcmcSxtMb40iWROqQBY-rTfJ7IX5YYD5QLW_up83i0XKZEN-FWkv6R68MzJ-EXrNkIiAENV2xkZCcukTriI1QUCYop7X0DS7VeqO_HgppkYoNKlalMZICcbZO-pHUZzn4?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/zX9nN76vunXRrEw8Z2IjU9NsG_FzmuWPqAXvWvCxLDQUNGb3m6TlCP8tfuti5_zDRPRZ7w6bV0lZLlzuDWxYYXlZn518qUoD9ICdNdGxXQlyxkDXhhBmWKymjgNovN8nlEIU-fjE9oXkwP6z1l1s1TwGNusZSaqq7LWmKB3FogdQqJ1l2HKattuGtu_7WpEQ?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/4NRR0989CX-ANZ3H-f0hkPTu7OmCSeHMj9gUXGE9NzAMOeL_Tf_shjehRILlfuefvzRddAJbhsF1yzV6aa9JBtp-4va4tjDd7dUfIj4OceETWN53MjNkInDAqyY3AmUMOwx5fvB0XljUDp0_aLLn5JaUaqy-cMusMEyu8DoSC2yGHht5r_L8t0tlypaljshm?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/HSuwceLGlOelBTLJPyJapbuDtVI8KDVU2S7VhiiLK9JgkOnhhPKJzz7uvEP-S8FwX_uBd_biRyOJIrChUoA4xgjUPL7r0vtCbzQ5Gr_1YSbOrvakyLqenq92Ii3BCFkAcyBL-lUhTTCkvVtmJ4SFT6Q2T_kG0Wdu57vpt4igcjfpFYwk3ZgTjysU7CCLd8M4?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/8FZSIV9l-4kLIypq-TcDMkRTQjcMTqTqx_IxM53nqsiq0VyJfzHRzCEIOg-ACm2ifKV-HqUWjlUONrGYXH-go8X8kRAX7ETFGYGzSsUJzgRM-voI2lu6MpdXrdqt3FphkEfoIll2Jb_37qSRtzptJ6iDpJpXGxa5TI4bCVLco_VLEIr8yyIpb2LqvSlz6kUA?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/sCWe4UTAIug3vUJR6B8Q6nC8UIpzBGyl0OsyI_W92m0YVF8T6QrT-oBge_pwY_LQrxzoutKmtXeeGxTht9Xw_S3AZONCr5JAW_-Uss4N1VCVeKS4FLvmiPprsz93G8vcNpyQ9oUTaSXdwW3yD-SsQ9gKfFEgItWcfTy-r9xejkpGf6XJgdKx6t1eSdfE3lp5?purpose=fullsize)

**Azure Container Apps** is a PaaS service for running containerized applications without having to manage the underlying Kubernetes infrastructure.

You provide:

* Container image
* CPU and memory
* Environment
* Ingress configuration
* Scaling rules
* Environment variables/secrets
* Optional storage and networking configuration

Azure manages the underlying infrastructure.

### Simple example

Suppose you have:

```text
Docker Image
    ↓
myapp:1.0
    ↓
Azure Container App
    ↓
https://myapp.example.com
```

You don't need to create:

```text
AKS Cluster
 ├── Node Pool
 │    ├── VM
 │    ├── VM
 │    └── VM
 ├── Kubernetes API Server
 ├── Pods
 ├── Services
 └── Ingress Controller
```

Azure handles that infrastructure for you.

---

# 2. Does Azure Container Apps use AKS in the background?

**Conceptually, yes, it is built on Kubernetes technology. But don't think of an Azure Container App as "my application running on my AKS cluster."**

This distinction is very important.

Azure Container Apps is built using technologies from the Kubernetes ecosystem, including:

* Kubernetes
* KEDA for event-driven autoscaling
* Dapr integration
* Envoy-based ingress architecture

But **Microsoft manages the underlying infrastructure and Kubernetes layer**.

You don't get:

* Kubernetes API server access
* Worker-node management
* Node-pool management
* `kubectl` access to an underlying cluster
* Control over Kubernetes nodes

So:

```text
AKS
────────────────────────────
You manage Kubernetes
You manage node pools
You manage cluster configuration
You use kubectl
You manage upgrades
You manage Kubernetes resources


Container Apps
────────────────────────────
Microsoft manages Kubernetes infrastructure
You manage the application
You don't manage nodes
You don't manage the Kubernetes cluster
```

A good interview answer is:

> **Azure Container Apps provides a serverless container platform built on Kubernetes technology, but Microsoft manages the Kubernetes infrastructure. It is not an AKS cluster that you manage.**

---

# 3. Container Apps vs ACI vs AKS

This is one of the most important comparisons.

| Feature                | ACI                             | Container Apps                 | AKS                              |
| ---------------------- | ------------------------------- | ------------------------------ | -------------------------------- |
| Service type           | Container instance              | Managed container app platform | Managed Kubernetes               |
| Kubernetes             | No                              | Kubernetes-based platform      | Full Kubernetes                  |
| Manage nodes           | No                              | No                             | Yes                              |
| Manage cluster         | No                              | No                             | Yes                              |
| Kubernetes API         | No                              | No                             | Yes                              |
| Autoscaling            | Limited                         | Excellent                      | Excellent                        |
| Scale to zero          | Yes, depending on usage pattern | Yes                            | Generally no                     |
| Ingress                | Basic networking                | Built-in HTTP/TCP ingress      | You configure it                 |
| Revisions              | No                              | Yes                            | Kubernetes deployments           |
| Traffic splitting      | No                              | Yes                            | Possible with Kubernetes tooling |
| KEDA                   | No                              | Built in                       | Can configure                    |
| Dapr                   | No                              | Integrated option              | Can install/configure            |
| Microservices          | Basic                           | Excellent                      | Excellent                        |
| Simple container       | Excellent                       | Good                           | Overkill                         |
| Kubernetes control     | None                            | None                           | Full                             |
| Operational complexity | Very low                        | Low                            | High                             |

---

# 4. ACI — Azure Container Instances

Think:

> **"I just want to run this container."**

For example:

```text
Docker Image
     ↓
ACI
     ↓
Container
     ↓
Application
```

You don't need to create a VM.

You don't need Kubernetes.

### Example

You have a temporary NGINX container:

```text
nginx:latest
```

You can run it in ACI.

This is useful for:

* Testing
* Short-lived jobs
* Simple container workloads
* CI/CD tasks
* Batch-style workloads

### ACI limitation

Suppose you build:

```text
Frontend
Backend
Worker
Queue
API
```

and you want:

* Autoscaling
* HTTP ingress
* revisions
* traffic splitting
* event-based scaling

ACI becomes less attractive.

That's where Container Apps comes in.

---

# 5. Azure Container Apps

Think:

> **"I want to run an application made from containers, but I don't want to manage Kubernetes."**

Example:

```text
                    Azure Container Apps Environment
                    ┌───────────────────────────────┐
                    │                               │
Internet ──→ Ingress ──→ Frontend Container App    │
                    │                               │
                    │          ↓                    │
                    │      Backend API              │
                    │          ↓                    │
                    │      Worker                   │
                    │                               │
                    └───────────────────────────────┘
```

This is particularly good for:

* Microservices
* APIs
* Web applications
* Background workers
* Event-driven applications
* Jobs
* Containerized applications that need autoscaling

---

# 6. AKS

Think:

> **"I need Kubernetes and I want control over Kubernetes."**

For example:

```text
AKS Cluster
│
├── Node Pool
│   ├── VM
│   ├── VM
│   └── VM
│
├── Namespace
│
├── Deployment
│
├── Pods
│
├── Service
│
├── Ingress
│
└── ConfigMap / Secret
```

You can use:

```bash
kubectl get pods
kubectl get nodes
kubectl get services
kubectl apply -f deployment.yaml
```

With Container Apps, you don't manage those Kubernetes objects directly.

---

# 7. Real-world comparison

Imagine you are building an online shopping application.

### Option 1 — ACI

```text
ACI
 └── Shopping Container
```

Good for a simple workload.

---

### Option 2 — Container Apps

```text
Container Apps Environment
│
├── Frontend
│
├── Product API
│
├── Order API
│
├── Payment API
│
└── Background Worker
```

Azure can automatically scale the applications.

For example:

```text
Normal traffic

Frontend:     1 replica
Product API:  2 replicas
Order API:    1 replica
Worker:       1 replica
```

During a sale:

```text
High traffic

Frontend:     10 replicas
Product API:  20 replicas
Order API:    15 replicas
Worker:       10 replicas
```

You don't manually add Kubernetes nodes.

---

### Option 3 — AKS

You can build the same architecture:

```text
AKS
│
├── Frontend Deployment
├── Product API Deployment
├── Order API Deployment
├── Payment API Deployment
│
├── Services
├── Ingress
├── HPA
├── KEDA
└── Nodes
```

But now **you are responsible for much more Kubernetes configuration and operations**.

---

# 8. Container Apps Environment

This is an important ACA concept.

A **Container Apps environment** is a secure boundary around one or more Container Apps.

Think of it as:

```text
Container Apps Environment
│
├── App 1
├── App 2
├── App 3
├── App 4
└── Jobs
```

Applications inside the same environment can communicate with each other.

### Example

```text
Production Environment
│
├── frontend
├── product-api
├── order-api
└── payment-api
```

You might have another environment:

```text
Development Environment
│
├── frontend-dev
├── product-api-dev
└── order-api-dev
```

This provides separation between environments.

---

# 9. Why create an Environment?

Suppose your company has:

```text
Development
Testing
Production
```

You can create:

```text
ACA Environment
       │
       ├── Dev Apps
       └── Dev Jobs
```

and:

```text
ACA Environment
       │
       ├── Test Apps
       └── Test Jobs
```

and:

```text
ACA Environment
       │
       ├── Production Apps
       └── Production Jobs
```

The environment provides a common infrastructure boundary for those applications.

---

# 10. Container App Profile

This is another important concept.

A **Container Apps environment can use workload profiles** to determine the compute characteristics available to applications.

Think:

```text
Container Apps Environment
        │
        ├── Consumption
        │
        ├── Dedicated workload profile
        │
        └── Different compute profiles
```

The exact available profiles depend on the environment/region and current Azure offerings.

### Consumption

Good for:

```text
Variable traffic
Short-lived workloads
Scale-out workloads
Pay-for-use scenarios
```

The platform dynamically allocates compute.

### Dedicated workload profiles

Useful when you need more predictable/dedicated compute characteristics.

For example:

```text
Production Environment

Workload Profile
       │
       ├── App A
       ├── App B
       └── App C
```

You can choose an appropriate workload profile for the workload rather than managing VM nodes yourself.

---

# 11. Ingress

Ingress determines:

> **How traffic reaches your Container App.**

This is extremely important.

Suppose:

```text
Internet
   ↓
HTTPS
   ↓
Container App
   ↓
Container
```

You can enable HTTP ingress.

For example:

```text
https://myapp.<azure-container-app-domain>
```

Azure provides the networking/ingress layer.

---

# 12. External vs Internal Ingress

There are two important concepts.

### External ingress

Application is accessible from outside the environment.

```text
Internet
   │
   ↓
External Ingress
   │
   ↓
Container App
```

Example:

```text
https://frontend.example.com
```

Use this for:

* Public websites
* Public APIs
* Internet-facing applications

---

### Internal ingress

Application is reachable only within the environment/network boundary.

```text
VNet
 │
 ├── App A
 │
 └── Internal Container App
```

For example:

```text
Frontend
   ↓
Internal Backend API
```

The backend doesn't need to be exposed directly to the Internet.

---

# 13. Very important: Ingress does not mean "load balancer that I manage"

In AKS you might explicitly configure:

```text
Ingress Controller
      ↓
Service
      ↓
Pods
```

In Container Apps, Azure provides the ingress capability.

You mainly configure:

```text
Ingress
 ├── Enabled/Disabled
 ├── External/Internal
 ├── Target port
 ├── Transport
 └── Traffic configuration
```

Azure handles the underlying implementation.

---

# 14. Target Port

Suppose your container application listens on:

```text
Port 8080
```

You configure:

```text
Ingress
   ↓
Target port = 8080
```

Traffic might look like:

```text
Client
  ↓
HTTPS : 443
  ↓
Container Apps ingress
  ↓
Container : 8080
```

You don't need to expose port 8080 directly to the Internet.

---

# 15. Container Apps Revision

This is one of the great features of Container Apps.

Suppose you have:

```text
myapp:v1
```

Then you deploy:

```text
myapp:v2
```

ACA can create:

```text
Revision 1
    ↓
myapp:v1

Revision 2
    ↓
myapp:v2
```

You can send traffic:

```text
Revision 1 → 90%
Revision 2 → 10%
```

Then gradually move:

```text
80 / 20
↓
50 / 50
↓
10 / 90
↓
0 / 100
```

This is useful for **blue/green and canary-style deployments**.

---

# 16. Scaling

This is where ACA becomes much more powerful than basic ACI.

Imagine:

```text
HTTP Requests
      ↓
Container App
```

At low traffic:

```text
1 replica
```

At high traffic:

```text
10 replicas
```

And when there is no traffic:

```text
0 replicas
```

depending on the scaling configuration.

Container Apps uses **KEDA** for event-driven autoscaling.

For example:

```text
Queue messages = 1000
        ↓
KEDA
        ↓
Increase replicas
        ↓
Worker processes messages
        ↓
Queue = 0
        ↓
Scale down
```

This is a major reason ACA is attractive for microservices and event-driven applications.

---

# 17. Simple decision tree

For your AZ-104 lesson, I would remember this:

```text
                 Container workload
                        │
             ┌──────────┴──────────┐
             │                     │
       Simple container       Application platform
             │                     │
            ACI             ┌──────┴──────┐
                            │             │
                       Don't need     Need Kubernetes
                       Kubernetes      control
                            │             │
                     Container Apps     AKS
```

### One-line memory trick

**ACI → Run a container**

**Container Apps → Run an application made of containers**

**AKS → Run and manage Kubernetes**

---

## 18. Real-world example

Imagine you are building a food-delivery platform.

### ACI

You need to run a temporary image-processing container:

```text
Order Image
    ↓
ACI
    ↓
Resize image
    ↓
Exit
```

Perfect use case.

### Container Apps

Your application has:

```text
Customer API
Restaurant API
Order API
Payment API
Notification Worker
```

You don't want to manage Kubernetes.

Use:

```text
Container Apps Environment
│
├── Customer API
├── Restaurant API
├── Order API
├── Payment API
└── Notification Worker
```

### AKS

Your enterprise needs:

* Kubernetes operators
* Custom controllers
* Helm
* Service mesh
* Fine-grained Kubernetes networking
* Custom node pools
* GPU workloads
* Kubernetes-native tooling
* Full Kubernetes API

Then:

```text
AKS
```

is more appropriate.

---

# 19. The key AZ-104 takeaway

If an interviewer asks:

**"Is Azure Container Apps AKS?"**

Don't answer simply **yes**.

Say:

> **Azure Container Apps is a managed application platform built on Kubernetes technology. Microsoft manages the underlying Kubernetes infrastructure, so customers don't manage the cluster, nodes, or Kubernetes API. AKS, on the other hand, gives customers direct control over a managed Kubernetes cluster.**

And:

> **ACI is primarily for running individual containers without Kubernetes, while Container Apps adds application-level capabilities such as ingress, autoscaling, revisions, traffic splitting, and microservice-oriented features.**

That distinction will make the three services much easier to remember.
