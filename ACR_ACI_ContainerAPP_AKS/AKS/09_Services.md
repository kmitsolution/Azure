## ClusterIP Service in Kubernetes (Concept + Example)

### What is ClusterIP?

A **ClusterIP Service** is the **default Service type** in Kubernetes.

👉 It provides:

* A **stable internal IP address**
* Access to your application **only within the cluster**

> It is mainly used for **communication between Pods** (internal traffic).

---

## Scenario

* You have a **Deployment** with **3 replicas (Pods)**
* Each Pod runs your application
* You want a **single stable endpoint** to access all Pods

👉 That’s where a **ClusterIP Service** comes in.

---

## Step-by-Step Conceptual Flow

### 1. Create a Deployment (3 replicas)

* Deployment creates:

  * 3 Pods
  * Each Pod gets its own IP (dynamic)

Example (conceptual YAML):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

👉 Now:

* 3 Pods are running
* But accessing them directly is unreliable (IPs can change)

---

### 2. Expose Deployment as a ClusterIP Service

You use the **expose command** to create a Service from the Deployment.

Conceptually, this generates a Service like:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: ClusterIP   # default
  selector:
    app: my-app
  ports:
  - port: 80        # Service port
    targetPort: 80  # Pod container port
```

👉 Important points:

* `selector` matches Pods with label `app: my-app`
* Service automatically connects to those Pods
* A **ClusterIP is assigned** (e.g., `10.96.x.x`)

---

### 3. Using `--dry-run=client -o yaml`

Instead of directly creating the Service, you can:

* Generate YAML first
* Review/edit it
* Then apply it

Concept:

```bash
kubectl expose deployment my-app \
  --port=80 --target-port=80 \
  --type=ClusterIP \
  --dry-run=client -o yaml > service.yaml
```

👉 What this does:

* Does NOT create the Service
* Outputs YAML definition
* Saves it into `service.yaml`

---

### 4. Apply the YAML

Then you create the Service using the YAML file:

```bash
kubectl apply -f service.yaml
```

👉 Now the ClusterIP Service is created.

---

## How ClusterIP Works Internally

* Kubernetes assigns a **virtual IP** (ClusterIP)
* Example: `10.96.45.12`
* This IP:

  * Is **internal only**
  * Is handled by **kube-proxy**

### Traffic Flow

```
Pod → Service (ClusterIP) → One of the 3 Pods
```

👉 Load balancing:

* Kubernetes distributes traffic across all 3 Pods
* Typically round-robin

---

## How to Access It

Inside the cluster, you can use:

* Service name: `my-app`
* Full DNS: `my-app.default.svc.cluster.local`

Example:

```
http://my-app
```

---

## Key Characteristics of ClusterIP

* Default Service type
* Internal access only (not reachable from outside)
* Stable IP even if Pods restart
* Automatically load balances traffic
* Uses label selectors to find Pods

---

## Simple Summary

* Deployment → creates 3 Pods
* Pods → dynamic IPs
* ClusterIP Service → gives **one stable IP + DNS name**
* All internal traffic goes through the Service

## NodePort Service in Kubernetes 

### What is NodePort?

A **NodePort Service** is used to expose your application **outside the cluster**.

👉 It opens a specific port on **every Node (VM/Server)** in the cluster.

> So anyone can access your app using:
> **`NodeIP : NodePort`**

---

## Scenario (Same as before)

* Deployment with **3 replicas**
* Each Pod runs your application
* Now we want to access it **from outside the cluster**

---

## Step-by-Step Conceptual Flow

### 1. Deployment (3 replicas)

Same as before:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

👉 Result:

* 3 Pods running
* Each has its own dynamic IP

---

### 2. Expose Deployment as NodePort Service

Instead of ClusterIP, we now use **NodePort**

Conceptually, this creates:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - port: 80          # Service port (inside cluster)
    targetPort: 80    # Pod port
    nodePort: 30007   # External port (on each Node)
```

👉 Important:

* `type: NodePort` enables external access
* `nodePort` is usually auto-assigned (30000–32767)
* You can also manually specify it (like `30007`)

---

### 3. Using `--dry-run=client -o yaml`

Generate YAML without creating the Service:

```bash
kubectl expose deployment my-app \
  --port=80 --target-port=80 \
  --type=NodePort \
  --dry-run=client -o yaml > service.yaml
```

👉 This:

* Creates YAML definition
* Saves it to `service.yaml`
* Does NOT create the Service yet

---

### 4. Apply the YAML

```bash
kubectl apply -f service.yaml
```

👉 Now NodePort Service is created.

---

## How NodePort Works Internally

NodePort actually includes **two layers**:

1. **ClusterIP (internal)**
2. **NodePort (external)**

👉 So every NodePort Service has:

* A **ClusterIP** (for internal access)
* A **NodePort** (for external access)

---

## Traffic Flow

### From Outside the Cluster:

```
User → NodeIP:NodePort → Service → One of the Pods
```

Example:

```
http://192.168.1.10:30007
```

👉 You can use **any node's IP** in the cluster.

---

### From Inside the Cluster:

```
Pod → ClusterIP → Pod
```

👉 Same as ClusterIP behavior internally.

---

## Key Characteristics of NodePort

* Exposes app on **every Node**
* Accessible from outside the cluster
* Uses port range: **30000–32767**
* Includes ClusterIP internally
* Basic load balancing across Pods

---

## Summary (ClusterIP vs NodePort)

| Feature       | ClusterIP     | NodePort                  |
| ------------- | ------------- | ------------------------- |
| Access scope  | Internal only | Internal + External       |
| External URL  | ❌ No          | ✅ Yes (NodeIP:Port)       |
| Port exposure | No            | Yes (30000–32767)         |
| Use case      | Internal apps | Testing / simple exposure |

---

## Simple Understanding

* Deployment → 3 Pods
* NodePort Service → opens a port on all Nodes
* External users → hit Node IP + port
* Kubernetes → routes traffic to Pods



## LoadBalancer Service in Kubernetes (EKS Example)

### What is LoadBalancer Service?

A **LoadBalancer Service** exposes your application **to the internet** using a **cloud provider’s load balancer**.

👉 In your case (EKS – Amazon EKS):

* Kubernetes automatically provisions an **AWS Elastic Load Balancer**
* You get a **public IP / DNS**
* Traffic is routed to your Pods

---

## Scenario (Same Example)

* Deployment with **3 replicas**
* Running inside an **EKS cluster**
* Need **external/public access**

---

## Step-by-Step Conceptual Flow

### 1. Deployment (3 replicas)

Same as before:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
```

👉 Result:

* 3 Pods created
* Each Pod has its own internal IP

---

### 2. Expose Deployment as LoadBalancer Service

Now we expose it using **LoadBalancer**

Conceptual YAML:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - port: 80          # Service port
    targetPort: 80    # Pod port
```

👉 Important:

* No need to specify `nodePort` (Kubernetes assigns it automatically)
* AWS will create a **Load Balancer**

---

### 3. Using `--dry-run=client -o yaml`

Generate YAML first:

```bash
kubectl expose deployment my-app \
  --port=80 --target-port=80 \
  --type=LoadBalancer \
  --dry-run=client -o yaml > service.yaml
```

👉 This:

* Generates YAML
* Saves it to `service.yaml`
* Does NOT create the Service yet

---

### 4. Apply the YAML

```bash
kubectl apply -f service.yaml
```

👉 Now Kubernetes:

* Creates the Service
* Talks to AWS
* AWS provisions a **Load Balancer**

---

## What Happens in EKS (Behind the Scenes)

When you create a LoadBalancer Service:

1. Kubernetes calls AWS APIs
2. AWS creates an **Elastic Load Balancer (ELB)**
3. ELB gets:

   * Public IP or DNS name
4. ELB forwards traffic to:

   * NodePort → Pods

---

## Traffic Flow

### External Access (Internet)

```
User → AWS Load Balancer → NodePort → Service → Pod
```

Example:

```
http://a1b2c3d4e5.elb.amazonaws.com
```

---

### Internal Access

```
Pod → ClusterIP → Pod
```

👉 Same internal behavior as ClusterIP

---

## Layers in LoadBalancer Service

A LoadBalancer Service automatically includes:

1. **ClusterIP** (internal communication)
2. **NodePort** (node-level access)
3. **External Load Balancer** (AWS ELB)

---

## Key Characteristics (EKS)

* Fully managed by AWS
* Provides **public access**
* Automatically provisions ELB
* Supports scaling and high availability
* Works with **security groups** and AWS networking

---

## Summary (All 3 Together)

| Feature     | ClusterIP     | NodePort         | LoadBalancer (EKS)       |
| ----------- | ------------- | ---------------- | ------------------------ |
| Access      | Internal      | Node IP          | Public Internet          |
| External IP | ❌ No          | ❌ (uses Node IP) | ✅ Yes (ELB DNS/IP)       |
| Complexity  | Low           | Medium           | High (cloud integration) |
| Use case    | Internal apps | Testing          | Production apps          |

---

## Simple Understanding

* Deployment → 3 Pods
* LoadBalancer Service → creates AWS ELB
* ELB → exposes app to internet
* Traffic → ELB → Nodes → Pods

