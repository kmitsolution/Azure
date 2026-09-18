1. **What AKS is and how its architecture works**
2. **How AKS differs from an on-premises Kubernetes environment**

That makes the concepts much easier for students to understand.

# AKS — Azure Kubernetes Service

![Image](https://images.openai.com/static-rsc-4/70d2rchSf86JCjw7cWUAdYV1tDwRBVyQHi5-yMaR8jPCjEFy8hY5F48GBM6aK1BK-NdXfuNsejFKZkzYCNllZOBsXR-E75i_0Yw2Br3a-FO5jbcDLvL1ZkrcLYaiThbyeVUUAlE6jfoqW1N3Q6vG-n7PZZvf25Nyg4Vy3__zRfm6sClyLRt_J3WfqO7ZnBGH?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/zSZqFLIFQcRGVDJe_pAbQO5ZmjxgkHillSpHiolrRTmycCGDL-_GCwhvDibpcW62ucjTyTZdZmYmuVM591LZqg-HIcDlCOgrY_XG_E48W226QTKIebkMLklrLOzI6N7OBJt7X1SnJ9gq6KXI9Z83q1LbaE_OW0Nx-O8sr_azHp3lsSGQvduPDLYq7NJv3z-Q?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/adLdWb52dMrhQFuYzwP8zU2VgXGGo4VrIyC3ghqLMlAkW6Jd0KvBW4-susYFVUrajmg33bBnLRArzG24zDfceMiQEZZmOorwUcDGpGLu8xzKi3MKnb3eyrAXcXz36fgshR1B4gKQRmqZkSddptBiEJEInDBf00ynklDnoZ_DURaHlOgJuIPf3G18qkeMNsW3?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/pZgEG0N0KcskysYqkgykRGNlR4dFCRp7_KfK6lhE0Ty6j9o6-4fqTvKslRmtVuMgHwxy4S7LEfJtQtyeXSq6Eiv2jaWMQaZjl36vLSL04qyJRdWV8MZQxVuF64dxB6CUKbVTf0iI1QbMRApwEKNN45yOoT-RUEYIVJz7Ef0LI7QoNfYnYLJc-ltO1kQjef2B?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ftwowocCR6grPJ6F0dA4xQN0_f4v3qtpDzEN0Ri3CmPp_KSBqU0x1q-HHJOGMqc6GyOVcZjLC9NNg2WQlOblbs45zI8Duo7RlYqK20RqdlXVU2eebeCnOkWUBuqcI7p3nRKNIojryKHNbaDKTUKqROeu9sMmtf0ZlyWOZeS_Ov-5eST_LocXNavBixJ_DgLN?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/Fy2rsrI0g7X7StO7k-HQNJSYd4IVf6CVVX6CV5Uk8A2SBc28w8WFjFkBWm0bumYA7IlqL1yaghMIwcgmz2J-TWNEyRm48NUIvdLCLZvFBO9mFyDeHCJpXgnqGeIMaWNSW0VrfFzwkh9-m77f6esxZhkmw0Je7Nd_K_rUQga8tP5VQxFUxyhPTQNkRzuQYM8i?purpose=fullsize)

## 1. What is AKS?

**AKS (Azure Kubernetes Service)** is Microsoft's managed Kubernetes service.

Kubernetes itself is an open-source container orchestration platform. It provides capabilities such as:

* Container deployment
* Scaling
* Service discovery
* Load balancing
* Self-healing
* Rolling updates
* Configuration and secret management
* Scheduling containers across machines

With AKS, Azure manages much of the Kubernetes control-plane infrastructure for you, while you primarily manage the **worker nodes, node pools, applications, networking, security and workloads**.

A simple way to think about it:

```text
                    Azure
                      |
                      v
             +------------------+
             |       AKS        |
             +------------------+
                /            \
               /              \
              v                v
       Control Plane       Worker Nodes
       (Managed by         (Node Pools)
          Azure)                |
                               Pods
                                |
                           Containers
```

---

# 2. AKS Architecture

At a high level, AKS has two major parts:

```text
                    AKS Cluster
                         |
          +--------------+--------------+
          |                             |
          v                             v
   Control Plane                  Node Pools
   Managed by Azure              Managed by You
          |                             |
   +------+------+                +-----+-----+
   |      |      |                |           |
  API   Scheduler Controller   Node 1      Node 2
 Server  Manager  Manager       |           |
                              Pods        Pods
```

The two major components are:

### Control Plane

Responsible for **managing the Kubernetes cluster**.

### Node Pool

Contains the **worker nodes where your applications actually run**.

---

# 3. What is the Control Plane?

The control plane is essentially the **brain of Kubernetes**.

It makes decisions about the cluster.

For example:

> "There are currently 3 replicas of my application. One pod has failed. I need to create another pod."

The control plane handles this type of decision.

Important Kubernetes control-plane components include:

### API Server

The **Kubernetes API Server** is the primary communication interface.

When you execute:

```bash
kubectl get pods
```

the communication is approximately:

```text
kubectl
   |
   v
API Server
   |
   v
Kubernetes Cluster
```

Similarly:

```bash
kubectl create deployment nginx --image=nginx
```

goes through the Kubernetes API.

---

### Scheduler

The scheduler decides:

> "Which worker node should run this Pod?"

For example:

```text
Pod
 |
 v
Scheduler
 |
 +---- Node 1
 |
 +---- Node 2  <--- selected
 |
 +---- Node 3
```

The scheduler considers things such as:

* Available CPU
* Available memory
* Node selectors
* Taints and tolerations
* Affinity/anti-affinity
* Other scheduling constraints

---

### Controller Manager

Controllers continuously compare:

**Desired state vs actual state**

Suppose you specify:

```yaml
replicas: 3
```

Kubernetes understands:

```text
Desired State = 3 Pods
```

But currently:

```text
Actual State = 2 Pods
```

The controller notices the difference and takes action:

```text
Desired = 3
Actual  = 2

       ↓

Create another Pod

       ↓

Desired = 3
Actual  = 3
```

This is one of the fundamental concepts of Kubernetes.

---

### etcd

In a traditional Kubernetes architecture, **etcd** is the distributed key-value store containing Kubernetes cluster state.

Conceptually:

```text
API Server
    |
    v
  etcd
    |
    +-- Cluster configuration
    +-- Kubernetes objects
    +-- Desired state
    +-- Cluster metadata
```

In **AKS**, Azure manages the control plane and its underlying components, so you don't manage the control-plane VMs or directly administer its etcd infrastructure.

---

# 4. What does "Managed Control Plane" mean?

This is one of the most important concepts for AKS.

In an on-prem Kubernetes environment, you might have:

```text
Physical/Virtual Servers

+-------------------------+
| Control Plane Server 1  |
| Control Plane Server 2  |
| Control Plane Server 3  |
+-------------------------+

+-------------------------+
| Worker Node 1           |
| Worker Node 2           |
| Worker Node 3           |
+-------------------------+
```

You are responsible for the infrastructure.

With AKS:

```text
                Azure
                  |
        +---------+---------+
        |                   |
        v                   v
  Azure-managed       Your subscription
  Control Plane             |
                            v
                       Node Pools
                            |
                       Worker Nodes
                            |
                           Pods
```

Azure manages the control plane.

You don't normally have to:

* Create control-plane VMs
* Patch control-plane operating systems
* Install Kubernetes manually
* Maintain etcd
* Configure API-server HA manually
* Replace failed control-plane infrastructure manually

This is the **managed Kubernetes** advantage.

---

# 5. What is a Node Pool?

Now we come to the **worker side**.

A node pool is a collection of Kubernetes worker nodes with a common configuration.

For example:

```text
AKS Cluster

Node Pool: system

+----------------+
| Node 1         |
| Standard_D4s_v5|
+----------------+

+----------------+
| Node 2         |
| Standard_D4s_v5|
+----------------+


Node Pool: user

+----------------+
| Node 3         |
| Standard_D8s_v5|
+----------------+

+----------------+
| Node 4         |
| Standard_D8s_v5|
+----------------+
```

The nodes are Azure VMs used as Kubernetes worker nodes.

---

# 6. What runs on Worker Nodes?

The actual application workloads run on the worker nodes.

For example:

```text
AKS
 |
 +-- Node Pool
       |
       +-- Node 1
       |     |
       |     +-- Pod
       |     |    |
       |     |    +-- Container
       |     |
       |     +-- Pod
       |
       +-- Node 2
             |
             +-- Pod
                  |
                  +-- Container
```

So remember:

> **Control Plane manages the cluster. Worker Nodes run the workloads.**

That's probably the simplest definition to give students.

---

# 7. Why do we need Node Pools?

Different applications may have different requirements.

For example:

```text
System workloads
      |
      v
System Node Pool
D4s_v5

Application workloads
      |
      v
User Node Pool
D8s_v5

GPU workloads
      |
      v
GPU Node Pool
NC-series
```

You can therefore have different node pools for different workloads.

For example:

```text
AKS Cluster

        |
        +---- System Node Pool
        |       |
        |       +-- Node
        |       +-- Node
        |
        +---- Application Node Pool
        |       |
        |       +-- Node
        |       +-- Node
        |
        +---- GPU Node Pool
                |
                +-- GPU Node
                +-- GPU Node
```

---

# 8. What is the Infrastructure Resource Group?

This is a concept that often confuses people when they first create AKS.

Suppose you create:

```text
Resource Group:
MY-AKS-RG
```

and then create an AKS cluster:

```text
AKS Cluster:
myakscluster
```

Azure creates another resource group, typically with a name similar to:

```text
MC_MY-AKS-RG_myakscluster_centralindia
```

This is commonly called the **node resource group** or **managed infrastructure resource group**.

The important point is:

> **You create the AKS resource, but Azure creates and manages many of the infrastructure resources required to operate the cluster.**

---

# 9. Why are there two Resource Groups?

Think about it like this.

### Resource Group 1 — Your Resource Group

```text
MY-AKS-RG
   |
   +-- AKS Cluster
```

This is where your primary AKS resource lives.

### Resource Group 2 — Managed Infrastructure

```text
MC_MY-AKS-RG_myakscluster_centralindia
   |
   +-- VM Scale Set
   +-- Managed Disk
   +-- NIC
   +-- Load Balancer
   +-- Public IP
   +-- Network resources
   +-- Other AKS infrastructure
```

The second resource group contains infrastructure resources associated with the AKS cluster.

---

# 10. What resources might you see in the Infrastructure Resource Group?

When you open the managed resource group, you may see resources such as:

### Virtual Machine Scale Set

For example:

```text
aks-systempool-xxxxx-vmss
```

This represents the underlying compute infrastructure for a node pool.

Instead of manually creating:

```text
VM1
VM2
VM3
```

AKS can use a VM Scale Set to manage the nodes.

Conceptually:

```text
VMSS
 |
 +-- VM / Node 1
 +-- VM / Node 2
 +-- VM / Node 3
```

---

## 11. Managed Disks

Worker nodes require storage.

You may see managed disks associated with the nodes.

Conceptually:

```text
Node
 |
 +-- OS Disk
 |
 +-- Data Disk(s)
```

The OS disk contains the operating system used by the node.

---

# 12. Network Interface / NIC

Azure VMs need network interfaces.

Conceptually:

```text
Node
 |
 +-- NIC
      |
      +-- Private IP
      |
      +-- Subnet
```

The NIC connects the VM/node to the Azure virtual network.

---

# 13. Load Balancer

If you expose a Kubernetes Service using:

```yaml
type: LoadBalancer
```

AKS can provision Azure networking infrastructure.

For example:

```text
Internet
    |
    v
Public IP
    |
    v
Azure Load Balancer
    |
    v
AKS Service
    |
    v
Pods
```

This is where Kubernetes and Azure infrastructure start working together.

---

# 14. Public IP

Depending on your configuration, you may see public IP resources.

For example:

```text
Internet
   |
Public IP
   |
Load Balancer
   |
AKS Service
   |
Pod
```

A public IP provides an externally reachable IP address.

---

# 15. Important: Don't manually modify everything in this Resource Group

This is an important operational concept.

The infrastructure resource group contains resources that are managed as part of the AKS cluster.

Therefore:

> **Don't treat the managed infrastructure resource group like a normal resource group where you freely modify or delete resources.**

For example, manually deleting a VMSS, NIC, disk, or networking resource can affect the AKS cluster.

Students should understand:

```text
Your responsibility
       |
       v
AKS configuration + workloads
       |
       v
Azure-managed infrastructure
```

---

# 16. AKS vs On-Premises Kubernetes

This is probably the most important comparison for understanding why AKS exists.

## On-Premises Kubernetes

Suppose your company has its own data center.

You might have:

```text
             Data Center

       +---------------------+
       | Control Plane 1     |
       | Control Plane 2     |
       | Control Plane 3     |
       +---------------------+

       +---------------------+
       | Worker Node 1       |
       | Worker Node 2       |
       | Worker Node 3       |
       +---------------------+

       +---------------------+
       | Storage             |
       | Network             |
       | Load Balancer       |
       | Firewall            |
       +---------------------+
```

Your organization is responsible for almost everything.

---

# 17. Responsibilities in On-Prem Kubernetes

You may need to manage:

### Hardware

```text
Physical Servers
CPU
RAM
Disk
Network Cards
```

### Virtualization

```text
VMware
Hyper-V
KVM
```

### Operating System

```text
Linux
OS patches
Security updates
Drivers
```

### Kubernetes

```text
API Server
Scheduler
Controller Manager
etcd
kubelet
container runtime
```

### Networking

```text
Switches
Routers
Firewalls
Load Balancers
DNS
```

### Storage

```text
SAN
NAS
CSI
Storage arrays
```

This creates significant operational responsibility.

---

# 18. AKS Changes This Model

With AKS:

```text
                  Azure
                    |
        +-----------+-----------+
        |                       |
        v                       v
   Azure-managed            Customer
   Control Plane          Managed Layer
                              |
                              v
                         Node Pools
                              |
                              v
                         Applications
```

Azure takes responsibility for much of the control-plane infrastructure.

You focus more on:

```text
Applications
    ↓
Pods
    ↓
Services
    ↓
Ingress
    ↓
Configuration
    ↓
Security
    ↓
Node Pools
    ↓
Scaling
```

---

# 19. Simple Responsibility Comparison

| Area                          | On-Prem Kubernetes       | AKS                                 |
| ----------------------------- | ------------------------ | ----------------------------------- |
| Physical hardware             | You manage               | Azure                               |
| Data center                   | You manage               | Azure                               |
| Control-plane infrastructure  | You manage               | Azure-managed                       |
| Kubernetes API server         | You manage               | Azure-managed                       |
| etcd                          | You manage               | Azure-managed                       |
| Control-plane OS              | You manage               | Azure                               |
| Worker nodes                  | You manage               | You manage/configure                |
| Node pools                    | You manage               | You configure/manage                |
| Application Pods              | You manage               | You manage                          |
| Kubernetes Services           | You manage               | You manage                          |
| Application deployment        | You manage               | You manage                          |
| Container images              | You manage               | You manage                          |
| Networking configuration      | You manage               | Shared responsibility               |
| Storage configuration         | You manage               | Shared responsibility               |
| Kubernetes version operations | You manage more directly | Azure provides managed capabilities |
| Scaling                       | You manage               | Azure provides automation/options   |

---

# 20. A Very Good Real-World Analogy

Imagine you want to run a restaurant.

### On-prem Kubernetes

You have to build everything:

```text
Buy building
      ↓
Electricity
      ↓
Water
      ↓
Kitchen
      ↓
Equipment
      ↓
Staff
      ↓
Restaurant
```

You are responsible for the infrastructure.

### AKS

Azure gives you the infrastructure platform.

```text
Azure
  |
  +-- Managed Kubernetes Control Plane
  |
  +-- Compute infrastructure
  |
  +-- Networking infrastructure
  |
  +-- Storage infrastructure
```

You focus more on running your applications.

---

# 21. Complete AKS Architecture

For your training, I recommend presenting the architecture like this:

```text
                         AZURE
                           |
                           |
                    +------+------+
                    |     AKS     |
                    +------+------+
                           |
              +------------+------------+
              |                         |
              |                         |
       CONTROL PLANE                NODE POOLS
       Azure Managed                Customer Managed
              |                         |
       +------+------+            +-----+------+
       |      |      |            |            |
      API  Scheduler Controller   System      User
     Server         Manager       Pool        Pool
       |                             |           |
       |                             |           |
       |                         +---+---+   +---+---+
       |                         |       |   |       |
       |                       Node    Node Node    Node
       |                         |       |   |       |
       |                         +---+---+   +---+---+
       |                             |
       |                            Pods
       |                             |
       |                         Containers
       |
       +---- Kubernetes API
```

And around the AKS cluster:

```text
                 Azure Subscription
                        |
                  Resource Group
                        |
                   AKS Resource
                        |
        +---------------+---------------+
        |                               |
        v                               v
   AKS Control Plane              Managed Node RG
   Azure Managed                       |
                                  +----+----+
                                  |         |
                                 VMSS     Networking
                                  |         |
                                 Nodes    LB / IP
                                  |
                                 Pods
```

---

# 22. The Key Concept Students Should Remember

I would summarize the entire lesson with these **five statements**:

### 1. AKS is managed Kubernetes

> Azure manages the Kubernetes control plane.

### 2. Control Plane = Brain

> It manages and controls the Kubernetes cluster.

### 3. Node Pool = Workers

> Node pools contain worker nodes where application workloads run.

### 4. Managed Infrastructure Resource Group = Azure's infrastructure

> Azure creates a managed resource group containing infrastructure resources required by the AKS cluster.

### 5. AKS vs On-Prem

> With on-prem Kubernetes, you manage the infrastructure and Kubernetes control plane yourself. With AKS, Azure manages the control plane, allowing you to focus more on worker nodes, workloads, networking, security and applications.

---

## 23. One Final Mental Model

If students remember only one diagram, use this:

```text
                         AKS
                          |
             +------------+------------+
             |                         |
             v                         v
       CONTROL PLANE              NODE POOLS
       Azure Managed              You Manage
             |                         |
       "Brain of K8s"             "Workers"
             |                         |
             |                    +----+----+
             |                    |         |
             |                   Node      Node
             |                    |         |
             |                   Pods      Pods
             |                    |         |
             |                Containers  Containers
             |
             |
       API Server
       Scheduler
       Controllers
       etcd (managed)
```

And outside that:

```text
              Azure Subscription
                       |
                  Your Resource Group
                       |
                    AKS Resource
                       |
                       |
              Managed Infrastructure
                 Resource Group
                       |
          +------------+------------+
          |            |            |
         VMSS        Disks       Networking
          |                         |
        Nodes                    LB / IP
          |
        Pods
```

This gives a very clean foundation before moving into **AKS cluster creation, node pools, networking, Azure CNI vs kubenet, ingress, identities, ACR integration, and autoscaling**.
