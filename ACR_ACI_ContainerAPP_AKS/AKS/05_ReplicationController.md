Below is a **structured DevOps training–style explanation of Replication Controller (RC)** that you can use in your **Kubernetes lecture notes or documentation**.

---

# Kubernetes **Replication Controller (RC)**

## 1. Introduction

A **Replication Controller (RC)** ensures that a specified number of **pod replicas are always running in the Kubernetes cluster**.

If a pod fails or gets deleted, the Replication Controller automatically **creates a new pod** to maintain the desired number of replicas.

Example:

```id="r1"}
Desired Pods = 3
Running Pods = 3
```

If one pod crashes:

```id="r2"}
Desired Pods = 3
Running Pods = 2
```

Replication Controller will automatically create a new pod to restore the count.

---

# 2. Why Replication Controller is Important

Replication Controller helps with:

* **High Availability**
* **Fault Tolerance**
* **Automatic Pod Recovery**
* **Load Distribution**

Example scenario:

```id="r3"}
Web Application
Replica Count = 3
```

If one pod fails, Kubernetes creates another pod automatically.

---

# 3. Replication Controller Architecture

Normal Pod:

```id="r4"}
Pod
```

With Replication Controller:

```id="r5"}
Replication Controller
       |
       |---- Pod 1
       |---- Pod 2
       |---- Pod 3
```

The Replication Controller continuously monitors pods.

---

# 4. Components of Replication Controller

A Replication Controller manifest contains three main sections.

| Section       | Purpose                   |
| ------------- | ------------------------- |
| metadata      | name and labels           |
| spec.replicas | desired number of pods    |
| spec.selector | identifies pods to manage |
| spec.template | pod definition            |

---

# 5. Example Replication Controller YAML

Create file:

```id="r6"}
rc-nginx.yaml
```

```yaml id="r7"}
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 3
  selector:
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

```yaml id="r8"}
replicas: 3
```

This means Kubernetes must maintain **3 running pods**.

---

### selector

```yaml id="r9"}
selector:
  app: web
```

This tells the Replication Controller which pods to monitor.

---

### template

```yaml id="r10"}
template:
```

This defines **how the pod should be created**.

---

# 7. Create Replication Controller

Run the command:

```bash id="r11"}
kubectl create -f rc-nginx.yaml
```

---

# 8. Verify Replication Controller

Check RC:

```bash id="r12"}
kubectl get rc
```

Example output:

```id="r13"}
NAME       DESIRED   CURRENT   READY
nginx-rc   3         3         3
```

---

# 9. Check Pods Created by RC

```bash id="r14"}
kubectl get pods
```

Example:

```id="r15"}
nginx-rc-abc12
nginx-rc-def34
nginx-rc-ghi56
```

Three pods will be running.

---

# 10. Verify Pod Placement

```bash id="r16"}
kubectl get pods -o wide
```

Example:

```id="r17"}
NAME              NODE
nginx-rc-abc12    worker1
nginx-rc-def34    worker2
nginx-rc-ghi56    worker1
```

Pods are distributed across nodes.

---

# 11. Test Self-Healing

Delete one pod:

```bash id="r18"}
kubectl delete pod nginx-rc-abc12
```

Now check pods again:

```bash id="r19"}
kubectl get pods
```

You will notice a **new pod is automatically created**.

Example:

```id="r20"}
nginx-rc-xyz78
```

This demonstrates **self-healing capability**.

---

# 12. Scale Replication Controller

Increase replicas.

```bash id="r21"}
kubectl scale rc nginx-rc --replicas=5
```

Check again:

```bash id="r22"}
kubectl get pods
```

Now there will be **5 pods running**.

---

# 13. Delete Replication Controller

Delete RC:

```bash id="r23"}
kubectl delete rc nginx-rc
```

All pods created by the RC will also be removed.

---

# 14. Limitations of Replication Controller

Replication Controller is **an older Kubernetes resource**.

Limitations:

* Only supports **equality-based selectors**
* Limited update capabilities

---

# 15. Modern Replacement

Replication Controller has been replaced by:

```id="r24"}
ReplicaSet
```

ReplicaSet is used by **Deployments**.

Current best practice:

```id="r25"}
Deployment → ReplicaSet → Pods
```

---

# 16. Replication Controller vs ReplicaSet

| Feature          | Replication Controller | ReplicaSet         |
| ---------------- | ---------------------- | ------------------ |
| Selector support | Equality only          | Advanced selectors |
| Rolling updates  | Limited                | Supported          |
| Modern usage     | Deprecated             | Recommended        |

---

# 17. Real World Use

Today Replication Controller is mainly used for:

* learning Kubernetes basics
* understanding pod replication
* legacy systems

Modern clusters typically use:

```id="r26"}
Deployments
ReplicaSets
```

---

# 18. Summary

Replication Controller ensures:

```id="r27"}
Desired number of pods are always running
```

Features:

* Self healing
* High availability
* Automatic pod recreation
* Basic scaling

Key command:

```id="r28"}
kubectl scale rc
```

---
