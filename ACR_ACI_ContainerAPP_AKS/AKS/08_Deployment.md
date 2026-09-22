
# Kubernetes **DaemonSet**

## 1. Introduction

A **DaemonSet** ensures that **a copy of a pod runs on every node (or selected nodes) in a Kubernetes cluster**.

Whenever a new node is added to the cluster, Kubernetes automatically creates a pod on that node.

Similarly, when a node is removed, the corresponding pod is also removed.

Concept:

```text
Cluster Nodes
   |
   |---- worker1 → Pod
   |---- worker2 → Pod
   |---- worker3 → Pod
```

Each node runs **exactly one copy of the pod**.

---

# 2. Why DaemonSet is Used

DaemonSets are commonly used for **node-level services**.

Examples include:

| Use Case        | Example                  |
| --------------- | ------------------------ |
| Log collection  | Fluentd                  |
| Monitoring      | Prometheus Node Exporter |
| Networking      | Calico / Flannel         |
| Security agents | Falco                    |
| Storage drivers | CSI plugins              |

These services must run **on every node**.

---

# 3. DaemonSet Architecture

```text
DaemonSet
     |
     |---- Pod on worker1
     |---- Pod on worker2
     |---- Pod on worker3
```

Unlike Deployments, DaemonSets do **not create multiple replicas** per node.

Instead:

```text
1 Pod per Node
```

---

# 4. Example DaemonSet YAML

Create file:

```text
daemonset-nginx.yaml
```

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemon
spec:
  selector:
    matchLabels:
      app: nginx-daemon
  template:
    metadata:
      labels:
        app: nginx-daemon
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

# 5. Create DaemonSet

Run:

```bash
kubectl apply -f daemonset-nginx.yaml
```

---

# 6. Verify DaemonSet

Check DaemonSet:

```bash
kubectl get daemonset
```

Example output:

```text
NAME           DESIRED   CURRENT   READY
nginx-daemon   3         3         3
```

---

# 7. Verify Pods Created

```bash
kubectl get pods
```

Example output:

```text
nginx-daemon-abc12
nginx-daemon-def34
nginx-daemon-ghi56
```

---

# 8. Verify Pod Placement

Check which nodes the pods are running on.

```bash
kubectl get pods -o wide
```

Example output:

```text
NAME               NODE
nginx-daemon-abc12 worker1
nginx-daemon-def34 worker2
nginx-daemon-ghi56 worker3
```

Each node has **one pod**.

---

# 9. Node Addition Behavior

If a new node joins the cluster:

```text
worker4 added
```

DaemonSet automatically creates a pod on that node.

```text
worker4 → nginx-daemon pod
```

---

# 10. Node Removal Behavior

If a node is removed:

```text
worker2 removed
```

The corresponding DaemonSet pod is also removed.

---

# 11. Running DaemonSet on Specific Nodes

DaemonSets can be restricted to specific nodes using:

* nodeSelector
* nodeAffinity

Example:

```yaml
nodeSelector:
  env: prod
```

This runs the DaemonSet **only on production nodes**.

---

# 12. Example: DaemonSet with NodeSelector

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemon
spec:
  selector:
    matchLabels:
      app: nginx-daemon
  template:
    metadata:
      labels:
        app: nginx-daemon
    spec:
      nodeSelector:
        env: prod
      containers:
      - name: nginx
        image: nginx
```

Now pods run **only on nodes labeled `env=prod`**.

---

# 13. DaemonSet vs Deployment

| Feature       | Deployment   | DaemonSet              |
| ------------- | ------------ | ---------------------- |
| Pods per node | Multiple     | One                    |
| Replica count | Configurable | Automatically per node |
| Use case      | Applications | Node services          |

---

# 14. Real World Example

Example DaemonSet:

```text
Fluentd Log Collector
```

Architecture:

```text
worker1 → fluentd pod
worker2 → fluentd pod
worker3 → fluentd pod
```

Logs from all containers are collected locally.

---

# 15. DaemonSet Update Strategy

DaemonSet supports **rolling updates**.

Example configuration:

```yaml
updateStrategy:
  type: RollingUpdate
```

Pods are updated **one node at a time**.

---

# 16. DaemonSet Workflow

```text
Create DaemonSet
       ↓
Scheduler detects nodes
       ↓
Pod created on each node
       ↓
New node joins → new pod created
```

---

# 17. Summary

DaemonSet ensures:

```text
A pod runs on every node in the cluster
```

Common uses:

* Logging agents
* Monitoring agents
* Networking plugins
* Security tools

Key commands:

```bash
kubectl get daemonset
kubectl get pods -o wide
```
