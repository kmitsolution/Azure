
---

# Kubernetes **ReplicaSet**

## 1. Introduction

A **ReplicaSet** ensures that a specified number of **pod replicas are always running in a Kubernetes cluster**.

If a pod crashes, fails, or is deleted, the ReplicaSet automatically **creates a new pod** to maintain the desired number of replicas.

Example:

```text
Desired Pods = 3
Running Pods = 3
```

If one pod fails:

```text
Desired Pods = 3
Running Pods = 2
```

ReplicaSet automatically creates another pod.

---

# 2. Why ReplicaSet is Important

ReplicaSet provides:

* High availability
* Self-healing
* Automatic pod recreation
* Scaling capability

Example scenario:

```text
Web application
Replica count = 3
```

If one pod fails, Kubernetes automatically replaces it.

---

# 3. ReplicaSet Architecture

```text
ReplicaSet
     |
     |---- Pod 1
     |---- Pod 2
     |---- Pod 3
```

ReplicaSet continuously monitors pods and ensures the desired number is running.

---

# 4. Components of ReplicaSet

ReplicaSet YAML contains the following sections:

| Section       | Description               |
| ------------- | ------------------------- |
| apiVersion    | API version               |
| kind          | Resource type             |
| metadata      | Name and labels           |
| spec.replicas | Desired number of pods    |
| spec.selector | Identifies pods to manage |
| spec.template | Pod definition            |

---

# 5. Example ReplicaSet YAML

Create file:

```text
replicaset-nginx.yaml
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

# 6. Important Fields Explained

### replicas

```yaml
replicas: 3
```

This ensures **3 pods must always run**.

---

### selector

```yaml
selector:
  matchLabels:
    app: web
```

ReplicaSet manages pods with label:

```text
app=web
```

---

### template

```yaml
template:
```

This defines the **pod specification** used to create pods.

---

# 7. Create ReplicaSet

Run:

```bash
kubectl create -f replicaset-nginx.yaml
```

---

# 8. Verify ReplicaSet

Check ReplicaSet:

```bash
kubectl get rs
```

Example output:

```text
NAME       DESIRED   CURRENT   READY
nginx-rs   3         3         3
```

---

# 9. Check Pods Created by ReplicaSet

```bash
kubectl get pods
```

Example:

```text
nginx-rs-abc12
nginx-rs-def34
nginx-rs-ghi56
```

ReplicaSet automatically created these pods.

---

# 10. Check Pod Placement

```bash
kubectl get pods -o wide
```

Example output:

```text
NAME            NODE
nginx-rs-abc12  worker1
nginx-rs-def34  worker2
nginx-rs-ghi56  worker1
```

Pods are distributed across worker nodes.

---

# 11. Test Self-Healing

Delete a pod:

```bash
kubectl delete pod nginx-rs-abc12
```

Now check pods again:

```bash
kubectl get pods
```

ReplicaSet automatically creates a **new pod**.

Example:

```text
nginx-rs-new78
```

---

# 12. Scaling ReplicaSet

Increase replicas:

```bash
kubectl scale rs nginx-rs --replicas=5
```

Check pods:

```bash
kubectl get pods
```

Now there will be **5 pods running**.

---

# 13. Delete ReplicaSet

```bash
kubectl delete rs nginx-rs
```

All pods created by the ReplicaSet will also be removed.

---

# 14. Advanced Label Selectors

ReplicaSet supports **set-based selectors**, which are more powerful than ReplicationController.

Example:

```yaml
matchExpressions:
- key: env
  operator: In
  values:
  - prod
  - staging
```

Supported operators:

| Operator     | Meaning              |
| ------------ | -------------------- |
| In           | Match values         |
| NotIn        | Exclude values       |
| Exists       | Label must exist     |
| DoesNotExist | Label must not exist |

---

# 15. ReplicaSet vs ReplicationController

| Feature       | ReplicationController | ReplicaSet  |
| ------------- | --------------------- | ----------- |
| API version   | v1                    | apps/v1     |
| Selector type | Equality only         | Set-based   |
| Flexibility   | Limited               | Advanced    |
| Modern usage  | Deprecated            | Recommended |

---

# 16. ReplicaSet vs Deployment

In modern Kubernetes:

```text
Deployment → ReplicaSet → Pods
```

Example:

```text
Deployment
    |
    |---- ReplicaSet
            |
            |---- Pods
```

Deployment manages ReplicaSets automatically.

---

# 17. Real World Usage

Today, ReplicaSets are usually **not created directly**.

Instead, they are managed by:

```text
Deployments
```

ReplicaSet works behind the scenes.

---

# 18. Summary

ReplicaSet ensures:

```text
Specified number of pods are always running
```

Features:

* Self-healing
* Automatic pod creation
* Scaling
* Advanced selectors

Key command:

```bash
kubectl scale rs
```

---

