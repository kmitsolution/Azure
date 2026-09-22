## Non-Persistent Storage in Kubernetes

---

# What is Non-Persistent Storage?

Non-persistent storage refers to storage that **does NOT survive permanently**.

👉 It depends on:

* Container lifecycle ❌
* Pod lifecycle ❌

So data can be lost when:

* Container restarts
* Pod is deleted

---

# Types of Non-Persistent Storage

### 1. Default Container Storage

* Data stored inside container filesystem
* Lost when container restarts

### 2. emptyDir Volume

* Created when Pod starts
* Deleted when Pod is deleted
* Survives container restart but NOT Pod deletion

---

# Case Study 1: Default Behavior (No Volume)

## Step 1: Create Pod with nginx

```bash
kubectl run mypod --image=nginx
```

---

## Step 2: Create Directory and File

```bash
kubectl exec -it mypod -- /bin/bash
```

Inside container:

```bash
mkdir /myapp
echo "Hello Kubernetes" > /myapp/myapp.java
ls /myapp
```

👉 Output:

```
myapp.java
```

---

## Step 3: Update Image (nginx → tomcat)

```bash
kubectl set image pod/mypod nginx=tomcat
```

👉 This will:

* Kill old container ❌
* Start new container ❌

---

## Step 4: Check File Again

```bash
kubectl exec -it mypod -- /bin/bash
ls /myapp
```

👉 Output:

```
No such file or directory ❌
```

---

## Conclusion (Default Storage)

* Data stored in container layer
* Lost after container restart

```text
Container restart → data lost ❌
```

---

# Case Study 2: Using emptyDir Volume

---

## Step 1: Create Pod with emptyDir

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /myapp
      name: myvolume
  volumes:
  - name: myvolume
    emptyDir: {}
```

---

## Step 2: Apply Pod

```bash
kubectl apply -f pod.yaml
```

---

## Step 3: Create File

```bash
kubectl exec -it mypod -- /bin/bash
```

Inside container:

```bash
echo "Hello Kubernetes" > /myapp/myapp.java
ls /myapp
```

👉 Output:

```
myapp.java
```

---

## Step 4: Update Image (nginx → tomcat)

```bash
kubectl set image pod/mypod nginx=tomcat
```

---

## Step 5: Check File Again

```bash
kubectl exec -it mypod -- /bin/bash
ls /myapp
```

👉 Output:

```
myapp.java  ✅
```

---

## Why Data Exists?

* Data stored in **emptyDir (Pod volume)**
* Volume is NOT deleted when container restarts

---

## Step 6: Delete Pod

```bash
kubectl delete pod mypod
```

---

## Step 7: Recreate Pod

```bash
kubectl apply -f pod.yaml
```

---

## Step 8: Check Again

```bash
kubectl exec -it mypod -- ls /myapp
```

👉 Output:

```
(empty) ❌
```

---

## Conclusion (emptyDir)

```text
Container restart → data persists ✅  
Pod deletion → data lost ❌
```

---

# Final Comparison

| Scenario          | Default Storage | emptyDir |
| ----------------- | --------------- | -------- |
| Container restart | ❌ Lost          | ✅ Exists |
| Image update      | ❌ Lost          | ✅ Exists |
| Pod deletion      | ❌ Lost          | ❌ Lost   |

---

# Final Understanding

* **Default container storage**
  → Completely temporary

* **emptyDir**
  → Temporary but better (Pod-level)

👉 Both are **non-persistent** because:

* Data does NOT survive Pod deletion

---
