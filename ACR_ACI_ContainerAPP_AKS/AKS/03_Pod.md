# AKS / Kubernetes — Pods

After creating the AKS cluster and understanding **Control Plane, Node Pools, and Worker Nodes**, the next fundamental Kubernetes concept is the **Pod**.

A Pod is the **smallest deployable unit in Kubernetes**.

---

# 1. What is a Pod?

A Pod is a Kubernetes object that represents **one or more containers that are deployed together**.

The most common case is:

```text
Pod
 |
 +-- Container
       |
       +-- Application
```

For example:

```text
AKS Cluster
    |
    +-- Node 1
          |
          +-- Pod
               |
               +-- nginx container
```

A Pod gets:

* Its own IP address
* Its own network namespace
* Shared storage volumes
* One or more containers

### Important

A Pod is **not the same thing as a container**.

Think of it as:

```text
Kubernetes
    |
    v
   Pod
    |
    +------ Container
    |
    +------ Container
```

Usually, a Pod contains **one application container**, but Kubernetes also supports multiple containers in the same Pod.

---

# 2. Why Does Kubernetes Use Pods?

Kubernetes doesn't directly manage individual containers.

Instead:

```text
Kubernetes
     |
     v
    Pod
     |
     v
 Container(s)
```

Therefore, when you tell Kubernetes:

> Run an NGINX container

Kubernetes actually creates an **NGINX Pod containing an NGINX container**.

---

# 3. Pod vs Node

Students often confuse these two.

A **Node** is a worker machine.

A **Pod** is a workload running on that machine.

```text
AKS Cluster
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

So:

> **Node = Worker machine**

> **Pod = Unit of workload running on the worker**

---

# 4. Create a Pod Using `kubectl run`

Let's create a simple NGINX Pod.

```bash
kubectl run nginx --image=nginx
```

Here:

```text
nginx
```

is the Pod name.

```text
--image=nginx
```

specifies the container image.

Kubernetes creates:

```text
Pod: nginx
 |
 +-- nginx container
       |
       +-- nginx image
```

---

# 5. Verify the Pod

Run:

```bash
kubectl get pods
```

Short form:

```bash
kubectl get po
```

Example:

```text
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          20s
```

### Understanding the output

```text
NAME
```

Pod name.

```text
READY
```

Number of ready containers / total containers.

For:

```text
1/1
```

it means:

```text
1 container ready
1 container total
```

---

# 6. `kubectl get po -o wide`

Now run:

```bash
kubectl get po -o wide
```

Example:

```text
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE
nginx   1/1     Running   0          2m    10.244.1.5   aks-agentpool-xxxxx-vmss000001
```

The `-o wide` option gives additional information.

Important fields:

```text
NAME
READY
STATUS
RESTARTS
AGE
IP
NODE
```

The most interesting fields for this lesson are:

### Pod IP

```text
10.244.1.5
```

This is the Pod's IP address.

### NODE

```text
aks-agentpool-xxxxx-vmss000001
```

This tells us **which worker node is running the Pod**.

This is a very useful command for demonstrating scheduling:

```text
Pod
 |
 +-- Running on --> Node 2
```

---

# 7. Check the Pod in Detail

You can also use:

```bash
kubectl describe pod nginx
```

This provides information about:

* Pod configuration
* Containers
* Image
* IP address
* Node
* Events
* Volumes
* Conditions
* Errors

The **Events** section at the bottom is particularly useful when troubleshooting.

---

# 8. Delete the Pod

Delete the Pod using:

```bash
kubectl delete pod nginx
```

Expected:

```text
pod "nginx" deleted
```

Verify:

```bash
kubectl get pods
```

The Pod will no longer exist.

---

# 9. Important Observation

We created the Pod directly:

```bash
kubectl run nginx --image=nginx
```

Then deleted it:

```bash
kubectl delete pod nginx
```

Kubernetes will **not recreate it**.

Why?

Because we created a standalone Pod.

There is no Deployment maintaining the desired number of replicas.

Later, when we learn Deployments:

```text
Deployment
     |
     v
 ReplicaSet
     |
     v
   Pods
```

if a Pod is deleted, Kubernetes can create another one.

---

# 10. Generate a Pod Manifest Using `--dry-run`

Instead of directly creating the Pod, we can ask `kubectl` to generate the YAML manifest.

Command:

```bash
kubectl run nginx \
  --image=nginx \
  --dry-run=client \
  -o yaml
```

This does **not create the Pod**.

It only generates the YAML.

---

# 11. What Does `--dry-run=client` Mean?

This part:

```bash
--dry-run=client
```

means:

> Generate the Kubernetes object locally without actually sending it to the Kubernetes API server.

Therefore:

```text
kubectl run
      |
      +-- dry-run
            |
            v
        Generate YAML
            |
            X
       Don't create Pod
```

---

# 12. What Does `-o yaml` Mean?

```bash
-o yaml
```

means:

> Display the generated Kubernetes object in YAML format.

So:

```bash
kubectl run nginx \
  --image=nginx \
  --dry-run=client \
  -o yaml
```

produces something similar to:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
```

The exact generated output can vary slightly with Kubernetes versions.

---

# 13. Save the Manifest to a File

Instead of displaying the YAML on the screen, redirect it to a file.

### Linux / Bash

```bash
kubectl run nginx \
  --image=nginx \
  --dry-run=client \
  -o yaml > nginx-pod.yaml
```

### PowerShell

```powershell
kubectl run nginx `
  --image=nginx `
  --dry-run=client `
  -o yaml | Out-File -Encoding utf8 nginx-pod.yaml
```

Now you have:

```text
nginx-pod.yaml
```

---

# 14. View the Manifest

Linux/Bash:

```bash
cat nginx-pod.yaml
```

PowerShell:

```powershell
Get-Content nginx-pod.yaml
```

The file contains the Pod definition.

---

# 15. Kubernetes Manifest Structure

A basic Pod manifest looks like:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
    - name: nginx
      image: nginx
```

Let's understand the important sections.

### `apiVersion`

```yaml
apiVersion: v1
```

Specifies the Kubernetes API version.

Pods use:

```text
v1
```

---

### `kind`

```yaml
kind: Pod
```

Specifies the Kubernetes object type.

---

### `metadata`

```yaml
metadata:
  name: nginx
```

Contains identifying information.

For example:

* Name
* Labels
* Annotations

---

### `spec`

```yaml
spec:
```

Defines the desired configuration of the Pod.

---

### `containers`

```yaml
containers:
```

Defines the containers that should run inside the Pod.

---

# 16. Create Pod Using `kubectl create -f`

Once the manifest exists:

```text
nginx-pod.yaml
```

you can create the Pod using:

```bash
kubectl create -f nginx-pod.yaml
```

Output:

```text
pod/nginx created
```

Verify:

```bash
kubectl get pods
```

---

# 17. What Does `kubectl create -f` Mean?

```bash
kubectl create -f nginx-pod.yaml
```

means:

> Create the Kubernetes object described in this YAML file.

The flow is:

```text
YAML File
    |
    v
kubectl create -f
    |
    v
Kubernetes API Server
    |
    v
Pod Created
```

---

# 18. `kubectl apply -f`

Another very important command is:

```bash
kubectl apply -f nginx-pod.yaml
```

`apply` is commonly used to **create or update** Kubernetes resources declaratively.

For example:

```text
YAML
 |
 v
kubectl apply
 |
 +---- Resource doesn't exist
 |          |
 |          v
 |       Create
 |
 +---- Resource exists
            |
            v
          Update
```

---

# 19. `create` vs `apply`

This is important for students.

| Command                       | Purpose                         |
| ----------------------------- | ------------------------------- |
| `kubectl create -f file.yaml` | Create resource                 |
| `kubectl apply -f file.yaml`  | Create or update resource       |
| `kubectl delete -f file.yaml` | Delete resource defined in file |

For day-to-day declarative Kubernetes management, `apply` is commonly used.

---

# 20. Delete Using Manifest

Instead of:

```bash
kubectl delete pod nginx
```

you can delete the resource using the YAML:

```bash
kubectl delete -f nginx-pod.yaml
```

Output:

```text
pod "nginx" deleted
```

So we have:

```text
Create:
kubectl create -f nginx-pod.yaml

Apply:
kubectl apply -f nginx-pod.yaml

Delete:
kubectl delete -f nginx-pod.yaml
```

---

# 21. Create a Multicontainer Pod

A Pod can contain more than one container.

Example:

```text
Pod
 |
 +-- Container 1
 |      |
 |      +-- nginx
 |
 +-- Container 2
        |
        +-- sidecar
```

Example YAML:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: multicontainer-pod

spec:
  containers:

    - name: nginx
      image: nginx

    - name: sidecar
      image: busybox
      command:
        - sh
        - -c
        - "while true; do echo Sidecar running; sleep 10; done"
```

Create it:

```bash
kubectl apply -f multicontainer-pod.yaml
```

Check:

```bash
kubectl get pods
```

You should see:

```text
NAME                 READY   STATUS
multicontainer-pod   2/2     Running
```

The `2/2` means:

```text
2 containers ready
2 containers total
```

---

# 22. Why Use Multiple Containers in a Pod?

Multiple containers in one Pod are useful when the containers need to work closely together.

A common pattern is a **sidecar container**.

For example:

```text
             Pod
              |
       +------+------+
       |             |
       v             v
   Application     Sidecar
   Container       Container
       |             |
       |             +-- Logging
       |
       +-- Application
```

Both containers share:

* Network namespace
* Pod IP
* Volumes when configured

They can communicate using:

```text
localhost
```

because they are in the same network namespace.

---

# 23. `kubectl exec`

`kubectl exec` allows us to execute a command inside a running container.

For example:

```bash
kubectl exec nginx -- nginx -v
```

This executes:

```text
nginx -v
```

inside the `nginx` container.

---

# 24. Get an Interactive Shell

For an NGINX container, try:

```bash
kubectl exec -it nginx -- /bin/bash
```

You may get:

```text
root@nginx:/#
```

Now you are inside the container.

Try:

```bash
ls
```

or:

```bash
hostname
```

or:

```bash
cat /etc/os-release
```

Exit:

```bash
exit
```

---

# 25. `exec` With Multicontainer Pods

This is very important.

Suppose:

```text
Pod
 |
 +-- nginx
 |
 +-- sidecar
```

If you run:

```bash
kubectl exec -it multicontainer-pod -- /bin/sh
```

Kubernetes needs to know which container you want.

Use:

```bash
kubectl exec -it multicontainer-pod \
  -c nginx \
  -- /bin/bash
```

Or:

```bash
kubectl exec -it multicontainer-pod \
  -c sidecar \
  -- /bin/sh
```

The `-c` option specifies the container.

---

# 26. Labels in Kubernetes Pods

Labels are key-value pairs attached to Kubernetes objects.

Example:

```yaml
metadata:
  name: nginx
  labels:
    app: nginx
    environment: dev
```

Here we have:

```text
app = nginx
environment = dev
```

Labels are extremely important in Kubernetes.

They are used for:

* Selecting Pods
* Services
* Deployments
* ReplicaSets
* Scheduling and organization
* Filtering resources

---

# 27. Create Pod With Labels

You can generate a Pod with labels using:

```bash
kubectl run nginx \
  --image=nginx \
  --labels="app=nginx,environment=dev"
```

Check:

```bash
kubectl get pods --show-labels
```

Example:

```text
NAME    READY   STATUS    LABELS
nginx   1/1     Running   app=nginx,environment=dev
```

---

# 28. Get Pods Using a Label Selector

Suppose:

```text
app=nginx
```

You can select those Pods:

```bash
kubectl get pods -l app=nginx
```

Or:

```bash
kubectl get pods --selector=app=nginx
```

This is the foundation of how Kubernetes Services find Pods.

Conceptually:

```text
Service
   |
   | selector: app=nginx
   |
   v
+------+-------+
|              |
v              v
Pod            Pod
app=nginx      app=nginx
```

---

# 29. Add Labels to an Existing Pod

You can also use:

```bash
kubectl label pod nginx environment=dev
```

Verify:

```bash
kubectl get pod nginx --show-labels
```

---

# 30. Common Pod Creation Errors

This is an important troubleshooting section for your students.

## Error 1 — ImagePullBackOff

Example:

```text
STATUS
ImagePullBackOff
```

Usually means Kubernetes cannot pull the container image.

Possible reasons:

* Image doesn't exist
* Wrong image name
* Wrong tag
* Private registry authentication problem
* Registry unavailable

For example:

```bash
kubectl run test \
  --image=invalid-image-xyz
```

Check:

```bash
kubectl describe pod test
```

Look at:

```text
Events
```

---

# 31. ErrImagePull

You may see:

```text
ErrImagePull
```

This means Kubernetes attempted to pull the image but failed.

Check:

```bash
kubectl describe pod <pod-name>
```

The Events section usually gives more information.

---

# 32. CrashLoopBackOff

Example:

```text
STATUS
CrashLoopBackOff
```

This usually means the container starts but repeatedly exits/crashes.

Check:

```bash
kubectl logs <pod-name>
```

For example:

```bash
kubectl logs nginx
```

For a previous crashed container:

```bash
kubectl logs nginx --previous
```

---

# 33. Pod Stuck in Pending

Example:

```text
STATUS
Pending
```

The Pod hasn't been successfully scheduled/started.

Possible reasons include:

* Insufficient CPU
* Insufficient memory
* No suitable node
* Taints/tolerations mismatch
* Node problems
* PVC/storage issues

Start troubleshooting with:

```bash
kubectl describe pod <pod-name>
```

Look at:

```text
Events
```

---

# 34. ContainerCreating

You may see:

```text
ContainerCreating
```

This means Kubernetes is still creating the container.

If it stays there for an unusually long time, check:

```bash
kubectl describe pod <pod-name>
```

Potential causes include:

* Image pulling
* Volume mounting problems
* Network issues
* Container runtime problems

---

# 35. CreateContainerConfigError

You may encounter:

```text
CreateContainerConfigError
```

Common causes include incorrect configuration such as:

* Missing Secret
* Missing ConfigMap
* Incorrect environment variable reference
* Invalid volume configuration

Use:

```bash
kubectl describe pod <pod-name>
```

---

# 36. CrashLoopBackOff vs ImagePullBackOff

Students often confuse these.

| Status                       | Typical meaning                           |
| ---------------------------- | ----------------------------------------- |
| `ImagePullBackOff`           | Kubernetes cannot pull the image          |
| `ErrImagePull`               | Image pull failed                         |
| `CrashLoopBackOff`           | Container starts but repeatedly crashes   |
| `Pending`                    | Pod cannot currently be scheduled/started |
| `ContainerCreating`          | Container is still being created          |
| `CreateContainerConfigError` | Container configuration problem           |

The most important troubleshooting commands are:

```bash
kubectl get pods
```

```bash
kubectl describe pod <pod-name>
```

```bash
kubectl logs <pod-name>
```

and for a specific container:

```bash
kubectl logs <pod-name> -c <container-name>
```

---

# 37. Useful Pod Commands — Quick Reference

### Create Pod

```bash
kubectl run nginx --image=nginx
```

### List Pods

```bash
kubectl get pods
```

### Short form

```bash
kubectl get po
```

### Detailed Pod information

```bash
kubectl get po -o wide
```

### Describe Pod

```bash
kubectl describe pod nginx
```

### Pod logs

```bash
kubectl logs nginx
```

### Execute command

```bash
kubectl exec nginx -- nginx -v
```

### Interactive shell

```bash
kubectl exec -it nginx -- /bin/bash
```

### Delete Pod

```bash
kubectl delete pod nginx
```

### Generate YAML

```bash
kubectl run nginx \
  --image=nginx \
  --dry-run=client \
  -o yaml
```

### Create from YAML

```bash
kubectl create -f nginx-pod.yaml
```

### Apply YAML

```bash
kubectl apply -f nginx-pod.yaml
```

### Delete from YAML

```bash
kubectl delete -f nginx-pod.yaml
```

### Show labels

```bash
kubectl get pods --show-labels
```

### Select by label

```bash
kubectl get pods -l app=nginx
```

### Add label

```bash
kubectl label pod nginx environment=dev
```

---

# 38. Complete Hands-On Flow

For your AKS lab, I would have students execute these commands in this order:

```text
1. Create Pod
      ↓
2. Check Pod
      ↓
3. Check Pod + Node
      ↓
4. Describe Pod
      ↓
5. Execute command inside Pod
      ↓
6. Check logs
      ↓
7. Delete Pod
      ↓
8. Generate YAML
      ↓
9. Create Pod from YAML
      ↓
10. Apply YAML
      ↓
11. Add labels
      ↓
12. Query using labels
      ↓
13. Create Multicontainer Pod
      ↓
14. Exec into specific container
      ↓
15. Delete using YAML
```

The key commands to remember are:

```bash
kubectl run
kubectl get po
kubectl get po -o wide
kubectl describe pod
kubectl logs
kubectl exec
kubectl delete pod
kubectl run --dry-run=client -o yaml
kubectl create -f
kubectl apply -f
kubectl delete -f
```

And the most important conceptual distinction is:

```text
                     AKS Cluster
                          |
                     Node Pool
                          |
                    Worker Node
                          |
                         Pod
                    +-----+-----+
                    |           |
               Container    Container
                 (app)       (sidecar)
```

**Pod is the Kubernetes unit that wraps the container(s).** The next logical step after this is **Deployment → ReplicaSet → Pods**, where you can demonstrate why we normally don't create production Pods directly with `kubectl run`.
