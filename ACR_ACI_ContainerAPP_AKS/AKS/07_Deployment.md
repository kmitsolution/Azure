
# Kubernetes **Deployment**

## 1. Introduction

A **Deployment** is a Kubernetes object used to manage **ReplicaSets and Pods**.

It provides:

* Pod management
* Rolling updates
* Rollback capability
* Self-healing
* Scaling

Deployment architecture:

```
Deployment
     |
     |---- ReplicaSet
             |
             |---- Pods
```

---

# 2. Example Deployment YAML

Create a deployment file.

### deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx:1.25
        ports:
        - containerPort: 80
```

---

# Create Deployment

```bash
kubectl apply -f deployment.yaml
```

Verify:

```bash
kubectl get deployments
kubectl get rs
kubectl get pods
```

---

# 3. Rolling Update Strategy

Rolling update is the **default deployment strategy** in Kubernetes.

It updates pods **gradually without downtime**.

Example process:

```
Old Version (v1) → New Version (v2)

Pod1 v1 → Pod1 v2
Pod2 v1 → Pod2 v2
Pod3 v1 → Pod3 v2
```

Users do not experience downtime.

---

# Rolling Update Example

Update image version:

```bash
kubectl set image deployment nginx-deployment nginx=nginx:1.26
```

Check rollout status:

```bash
kubectl rollout status deployment nginx-deployment
```

View rollout history:

```bash
kubectl rollout history deployment nginx-deployment
```

Rollback if needed:

```bash
kubectl rollout undo deployment nginx-deployment
```

---

# Rolling Update Configuration

Deployment supports parameters like:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

Explanation:

| Parameter      | Meaning                                |
| -------------- | -------------------------------------- |
| maxUnavailable | Maximum pods unavailable during update |
| maxSurge       | Extra pods created during update       |

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 2
```

---

# 4. Blue-Green Deployment

Blue-Green deployment is a **zero-downtime release strategy**.

Two environments exist:

```
Blue → Current Production
Green → New Version
```

Traffic is switched when the new version is ready.

Example:

```
Blue Environment (v1)
Green Environment (v2)
```

---

# Blue-Green Example

### Blue Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
      version: blue
  template:
    metadata:
      labels:
        app: web
        version: blue
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

---

### Green Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
      version: green
  template:
    metadata:
      labels:
        app: web
        version: green
    spec:
      containers:
      - name: nginx
        image: nginx:1.26
```

---

# Service Switch

Service initially points to **Blue**.

```yaml
selector:
  app: web
  version: blue
```

After testing Green:

```
Update service selector → version=green
```

Traffic switches instantly.

---

# Blue-Green Deployment Flow

```
Deploy Blue (v1)
       |
Deploy Green (v2)
       |
Test Green
       |
Switch Service Traffic
       |
Remove Blue
```

---

# 5. Canary Deployment

Canary deployment releases the new version to **a small subset of users first**.

Example:

```
90% users → Old Version
10% users → New Version
```

If stable, rollout continues.

---

# Canary Example

### Old Version Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-v1
spec:
  replicas: 5
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: app
        image: nginx:1.25
```

---

### Canary Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: v2
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
      - name: app
        image: nginx:1.26
```

---

# Service for Canary

```yaml
selector:
  app: myapp
```

Service sends traffic to **both deployments**.

Example distribution:

```
5 pods v1
1 pod v2
```

About **16% traffic goes to v2**.

---

# Canary Deployment Flow

```
Deploy v1
      |
Deploy v2 (small %)
      |
Monitor
      |
Increase replicas
      |
Full rollout
```

---

# 6. Comparison of Strategies

| Strategy       | Description                     | Downtime |
| -------------- | ------------------------------- | -------- |
| Rolling Update | Gradual replacement             | None     |
| Blue-Green     | Switch environments             | None     |
| Canary         | Test new version with few users | None     |

---

# 7. Real-World Use

| Company            | Strategy       |
| ------------------ | -------------- |
| Netflix            | Canary         |
| Amazon             | Blue-Green     |
| Kubernetes default | Rolling Update |

---

# 8. Summary

Deployment provides:

```
Pod management
Rolling updates
Rollback capability
Scaling
High availability
```

Deployment strategies:

```
Rolling Update
Blue-Green Deployment
Canary Deployment
```

---
