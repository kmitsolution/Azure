## 🧩 Namespaces in Kubernetes (Beginner Friendly)

---

# ❓ What is a Namespace?

A **Namespace** in Kubernetes is a way to **logically divide a cluster**.

👉 Think of it like folders:

```text
Cluster
 ├── default
 ├── kube-system
 ├── dev-namespace
 ├── prod-namespace
```

---

## 🧠 Why Namespaces?

* Separate environments (dev, test, prod)
* Avoid name conflicts
* Apply resource limits
* Manage access (RBAC)

---

# ⚙️ How Namespaces Work

👉 Resources inside a namespace are **isolated**

Example:

```text
dev-namespace → pod1
prod-namespace → pod1 (same name allowed)
```

✔ Same resource names can exist in different namespaces

---

# 🚀 Step 1: Create Namespace (Command)

```bash
kubectl create namespace dev-namespace
```

---

## 🔎 Verify

```bash
kubectl get namespaces
```

---

# 📄 Step 2: Create Namespace (YAML Manifest)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-namespace
```

---

## Apply:

```bash
kubectl apply -f namespace.yaml
```

---

# 📦 Step 3: Create Resource Inside Namespace

---

## Option 1: Using Command

```bash
kubectl run mypod --image=nginx -n dev-namespace
```

👉 This creates Pod inside `dev-namespace`

---

## Verify:

```bash
kubectl get pods -n dev-namespace
```

---

## Option 2: Using YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: dev-namespace
spec:
  containers:
  - name: nginx
    image: nginx
```

---

## Apply:

```bash
kubectl apply -f pod.yaml
```

---

# 🔍 How to Check Resources in Namespace

---

## List Pods in Specific Namespace

```bash
kubectl get pods -n dev-namespace
```

---

## List All Namespaces

```bash
kubectl get ns
```

---

## List All Resources in Namespace

```bash
kubectl get all -n dev-namespace
```

---

# ⚡ Important Commands

---

## Set Default Namespace (Optional)

```bash
kubectl config set-context --current --namespace=dev-namespace
```

👉 Now you don’t need `-n dev-namespace`

---

## Get Pods from All Namespaces

```bash
kubectl get pods -A
```

---

# 🧠 Key Points

* Namespace is **logical isolation**, not physical
* Default namespace = `default`
* Some namespaces are system-created:

  * `kube-system`
  * `kube-public`

---

# 🎯 Simple Flow

```text
Create Namespace → Deploy Resources → Access via Namespace
```

---

# 🧾 Interview-Level Answer

> “Namespaces in Kubernetes are used to logically isolate resources within a cluster. They help organize environments like dev and prod, avoid naming conflicts, and control access and resource usage.”

---

# 👍 Final Summary

* Namespace = logical separation
* Create via command or YAML
* Use `-n` flag to target namespace
* Resources are isolated inside namespace

---
