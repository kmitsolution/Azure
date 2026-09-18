# AKS — Pod Assignment to Worker Nodes

After learning Pods and Node Pools, the next important question is:

> **How does Kubernetes decide which worker node should run a Pod?**

By default, the **Kubernetes Scheduler** decides where a Pod should run.

For example:

```text
                    AKS Cluster
                         |
              +----------+----------+
              |                     |
              v                     v
           Node 1                 Node 2
        agentpool             agentpool
              |                     |
              |                     |
           Pod A                  Pod B
```

The scheduler considers available resources and various scheduling rules.

But sometimes we want to control **where a particular Pod should run**.

Kubernetes provides several mechanisms:

1. `nodeName`
2. `nodeSelector`
3. Node Affinity
4. Taints and Tolerations

A very important distinction is:

```text
nodeName
    ↓
"Put this Pod on this exact node"

nodeSelector
    ↓
"Put this Pod on a node having this label"

Node Affinity
    ↓
"Put this Pod on a node satisfying these rules"

Taints + Tolerations
    ↓
"Keep Pods away from this node unless they are allowed"
```

---

# 1. First See Your AKS Nodes

Run:

```bash
kubectl get nodes -o wide
```

Example:

```text
NAME                                STATUS   ROLES
aks-agentpool-12345678-vmss000000   Ready    <none>
aks-agentpool-12345678-vmss000001   Ready    <none>
```

Let's give these nodes meaningful labels for our demonstration.

> **Do not manually rename AKS node names.** We will use their existing names and add our own labels.

---

# 2. See Existing Node Labels

Run:

```bash
kubectl get nodes --show-labels
```

You will see many labels automatically created by AKS/Kubernetes.

For a cleaner view:

```bash
kubectl get nodes --label-columns=kubernetes.io/hostname
```

You can also get the exact node names:

```bash
kubectl get nodes -o name
```

---

# 3. Method 1 — `nodeName`

`nodeName` is the simplest way to force a Pod onto a **specific node**.

Suppose:

```text
Node 1 = aks-agentpool-xxxxx-vmss000000
Node 2 = aks-agentpool-xxxxx-vmss000001
```

We want our Pod to run specifically on Node 1.

Create:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-nodename

spec:
  nodeName: aks-agentpool-xxxxx-vmss000000

  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f nginx-nodename.yaml
```

Check:

```bash
kubectl get pod nginx-nodename -o wide
```

You should see:

```text
NAME             READY   STATUS    NODE
nginx-nodename   1/1     Running   aks-agentpool-xxxxx-vmss000000
```

The Pod is directly assigned to that node.

---

# 4. Why `nodeName` Is Usually Not Recommended

Although `nodeName` is simple, it is very rigid.

You are saying:

> "I don't care about scheduling rules. Put this Pod on this exact node."

If that node doesn't exist or isn't available:

```text
Pod
 |
 +-- nodeName = Node 1
                  |
                  X
              unavailable
```

The Pod won't automatically choose another suitable node.

Therefore, `nodeName` is mainly useful for:

* Testing
* Debugging
* Special scenarios
* Demonstrations

For normal workloads, **nodeSelector or node affinity** is generally more appropriate.

---

# 5. Method 2 — `nodeSelector`

`nodeSelector` is a much more practical mechanism.

Instead of specifying the node name, we specify a **label**.

For example:

```text
workload=app
```

We can add this label to Node 1.

First identify the node:

```bash
kubectl get nodes
```

Then:

```bash
kubectl label node <NODE1> workload=app
```

For example:

```bash
kubectl label node aks-agentpool-xxxxx-vmss000000 workload=app
```

Verify:

```bash
kubectl get nodes --show-labels
```

---

# 6. Create Pod Using `nodeSelector`

Now create:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-nodeselector

spec:

  nodeSelector:
    workload: app

  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f nginx-nodeselector.yaml
```

Check:

```bash
kubectl get pod nginx-nodeselector -o wide
```

The Pod should be scheduled to the node having:

```text
workload=app
```

---

# 7. How `nodeSelector` Works

The scheduler looks at:

```text
Pod
 |
 | nodeSelector:
 |   workload: app
 |
 v
Scheduler
 |
 +---- Node 1
 |       workload=app   ✓
 |
 +---- Node 2
         workload=db    X
```

Therefore:

```text
Pod → Node 1
```

The important concept is:

> **nodeSelector selects a node based on labels, not node names.**

---

# 8. Method 3 — Node Affinity

`nodeSelector` is simple but has limited expression capabilities.

For more sophisticated rules, use **Node Affinity**.

Node affinity allows you to express things like:

> Run this Pod on a node where `workload=app`.

or:

> Run this Pod on a node where `environment` is either `dev` or `test`.

or:

> Prefer a node with a particular label, but don't absolutely require it.

---

# 9. Required Node Affinity

Let's create a node label:

```bash
kubectl label node <NODE1> workload=app
```

Now create:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-affinity

spec:

  affinity:

    nodeAffinity:

      requiredDuringSchedulingIgnoredDuringExecution:

        nodeSelectorTerms:

          - matchExpressions:

              - key: workload
                operator: In
                values:
                  - app

  containers:

    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f nginx-affinity.yaml
```

Check:

```bash
kubectl get pod nginx-affinity -o wide
```

---

# 10. Understanding `requiredDuringSchedulingIgnoredDuringExecution`

This long name looks complicated, but break it down.

```text
required
    |
    v
The rule MUST be satisfied
```

```text
DuringScheduling
    |
    v
The rule is checked when Kubernetes schedules the Pod
```

```text
IgnoredDuringExecution
    |
    v
If the label changes after the Pod is already running,
Kubernetes does not automatically evict the Pod just because
the affinity rule is no longer satisfied.
```

So:

```text
requiredDuringSchedulingIgnoredDuringExecution
```

means approximately:

> **The rule is mandatory when scheduling the Pod.**

---

# 11. Node Affinity Operators

Node affinity supports operators such as:

```text
In
NotIn
Exists
DoesNotExist
Gt
Lt
```

For example:

```yaml
matchExpressions:
  - key: environment
    operator: In
    values:
      - production
```

means:

```text
environment = production
```

Another example:

```yaml
matchExpressions:
  - key: environment
    operator: NotIn
    values:
      - development
```

means:

```text
environment != development
```

---

# 12. Multiple Values

Suppose nodes have:

```text
environment=dev
environment=test
environment=prod
```

You can say:

```yaml
matchExpressions:
  - key: environment
    operator: In
    values:
      - dev
      - test
```

Meaning:

```text
environment = dev
       OR
environment = test
```

---

# 13. Preferred Node Affinity

There are two major affinity concepts you should teach:

### Required

```text
requiredDuringSchedulingIgnoredDuringExecution
```

The condition is mandatory.

### Preferred

```text
preferredDuringSchedulingIgnoredDuringExecution
```

The condition is a preference.

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-preferred

spec:

  affinity:

    nodeAffinity:

      preferredDuringSchedulingIgnoredDuringExecution:

        - weight: 100

          preference:

            matchExpressions:

              - key: workload
                operator: In
                values:
                  - app

  containers:

    - name: nginx
      image: nginx
```

Here Kubernetes says:

> "I prefer a node with `workload=app`, but if such a node isn't available, another suitable node may be selected."

This is different from:

```text
required
```

where the condition must be satisfied.

---

# 14. `nodeSelector` vs Node Affinity

| Feature                  | nodeSelector | Node Affinity |
| ------------------------ | ------------ | ------------- |
| Simple node selection    | Yes          | Yes           |
| Exact label match        | Yes          | Yes           |
| `In` / `NotIn`           | Limited      | Yes           |
| Multiple expressions     | Limited      | Yes           |
| Required rules           | Yes          | Yes           |
| Preferred rules          | No           | Yes           |
| Complex scheduling rules | No           | Yes           |
| Easy to learn            | Very easy    | More advanced |

Think of it as:

```text
nodeSelector
    ↓
Simple requirement

Node Affinity
    ↓
Advanced requirement/preference
```

---

# 15. Method 4 — Taints and Tolerations

Now we come to a concept that is slightly different.

`nodeSelector` and affinity answer:

> **Which nodes should this Pod run on?**

Taints answer:

> **Which Pods should NOT run on this node?**

This is a very important distinction.

---

# 16. Taint a Node

Suppose we have:

```text
Node 1
```

and we want to reserve it for special workloads.

Add a taint:

```bash
kubectl taint nodes <NODE1> dedicated=special:NoSchedule
```

For example:

```bash
kubectl taint nodes aks-agentpool-xxxxx-vmss000000 dedicated=special:NoSchedule
```

Now the node has:

```text
dedicated=special:NoSchedule
```

---

# 17. What Does `NoSchedule` Mean?

It means:

> Kubernetes should not schedule Pods that do not tolerate this taint onto the node.

Conceptually:

```text
                 Node 1
           dedicated=special
                  |
              NoSchedule
                  |
         +--------+--------+
         |                 |
      Pod A             Pod B
   No toleration     Has toleration
         |                 |
         X                 ✓
      Rejected           Allowed
```

---

# 18. Create a Pod Without Toleration

Try:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-no-toleration

spec:
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f nginx-no-toleration.yaml
```

If all suitable nodes are tainted in a way that excludes this Pod, it may remain:

```text
Pending
```

Check:

```bash
kubectl get pod nginx-no-toleration
```

Then:

```bash
kubectl describe pod nginx-no-toleration
```

Look at the **Events** section.

You may see scheduling messages indicating that nodes have taints the Pod doesn't tolerate.

---

# 19. Add a Toleration

Now we tell the Pod:

> "I am allowed to run on a node having the `dedicated=special` taint."

Example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx-toleration

spec:

  tolerations:

    - key: dedicated
      operator: Equal
      value: special
      effect: NoSchedule

  containers:

    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f nginx-toleration.yaml
```

Check:

```bash
kubectl get pod nginx-toleration -o wide
```

The Pod is now **allowed** to run on the tainted node.

---

# 20. Very Important: Toleration Does NOT Force Scheduling

This is one of the most common misconceptions.

Suppose:

```text
Node 1:
dedicated=special:NoSchedule

Pod:
tolerates dedicated=special
```

This means:

```text
Pod is ALLOWED on Node 1
```

It does **not** mean:

```text
Pod MUST run on Node 1
```

The scheduler can still choose another node.

If you want to say:

> "This Pod should run on the special node."

you normally combine:

```text
Taint + Toleration
        +
Node Affinity / nodeSelector
```

For example:

```text
Taint
dedicated=special:NoSchedule

       +

Toleration
dedicated=special

       +

Node Selector
dedicated=special
```

Now you have:

```text
Only special Pods
        |
        v
Special Node
```

---

# 21. Taints and Tolerations — Real-World Example

Imagine you have a GPU node:

```text
GPU Node
   |
   +-- Expensive GPU hardware
```

You don't want ordinary applications accidentally consuming that node.

You can taint it:

```bash
kubectl taint nodes <GPU-NODE> gpu=true:NoSchedule
```

Now ordinary Pods won't be scheduled there.

A GPU application can have:

```yaml
tolerations:
  - key: gpu
    operator: Equal
    value: "true"
    effect: NoSchedule
```

Now GPU workloads are allowed onto that node.

Usually you'd also add affinity/selector so the GPU workload actually targets GPU nodes.

---

# 22. AKS System Node Pool Example

This concept becomes especially useful when you have different AKS node pools.

For example:

```text
AKS Cluster
    |
    +-- System Node Pool
    |       |
    |       +-- Node
    |       +-- Node
    |
    +-- User Node Pool
            |
            +-- Node
            +-- Node
```

You may want application workloads to use the user node pool rather than system nodes.

Labels and scheduling rules help Kubernetes make that distinction.

---

# 23. Four Methods Compared

This is the table I would recommend for your training.

| Method           | Meaning                 | Example                        |
| ---------------- | ----------------------- | ------------------------------ |
| `nodeName`       | Exact node              | Run on Node 1                  |
| `nodeSelector`   | Match node label        | `workload=app`                 |
| Node Affinity    | Advanced node selection | Prefer/require certain labels  |
| Taint/Toleration | Restrict nodes          | Only allowed Pods can use node |

---

# 24. Think of Them as Questions

A very easy way to remember them:

### `nodeName`

> **Which exact node?**

```yaml
nodeName: node1
```

---

### `nodeSelector`

> **Which label must the node have?**

```yaml
nodeSelector:
  workload: app
```

---

### Node Affinity

> **What node-selection rules should I use?**

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
```

---

### Taint/Toleration

> **Which Pods are allowed onto this node?**

Node:

```bash
kubectl taint nodes node1 dedicated=special:NoSchedule
```

Pod:

```yaml
tolerations:
  - key: dedicated
    operator: Equal
    value: special
    effect: NoSchedule
```

---

# 25. Complete Hands-On Lab

For your AKS training, I suggest doing this practical exercise.

First:

```bash
kubectl get nodes
```

Assume:

```text
NODE1=aks-agentpool-xxxxx-vmss000000
NODE2=aks-agentpool-xxxxx-vmss000001
```

---

### Step 1 — Label Node 1

```bash
kubectl label node <NODE1> workload=app
```

Verify:

```bash
kubectl get nodes --show-labels
```

---

### Step 2 — Create Pod Using `nodeSelector`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-selector
spec:
  nodeSelector:
    workload: app
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f pod-selector.yaml
```

Verify:

```bash
kubectl get pod pod-selector -o wide
```

---

### Step 3 — Taint Node 1

```bash
kubectl taint node <NODE1> dedicated=special:NoSchedule
```

---

### Step 4 — Create Pod Without Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-no-toleration
spec:
  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f pod-no-toleration.yaml
```

Check:

```bash
kubectl get pod pod-no-toleration -o wide
```

Then:

```bash
kubectl describe pod pod-no-toleration
```

---

### Step 5 — Create Pod With Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-toleration

spec:
  tolerations:
    - key: dedicated
      operator: Equal
      value: special
      effect: NoSchedule

  containers:
    - name: nginx
      image: nginx
```

Apply:

```bash
kubectl apply -f pod-toleration.yaml
```

Verify:

```bash
kubectl get pod pod-toleration -o wide
```

---

# 26. Combining Toleration + Node Selector

For a very good final demonstration, combine both.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: special-app

spec:

  nodeSelector:
    workload: app

  tolerations:

    - key: dedicated
      operator: Equal
      value: special
      effect: NoSchedule

  containers:

    - name: nginx
      image: nginx
```

Now:

```text
Node 1
  |
  +-- workload=app
  |
  +-- dedicated=special:NoSchedule
  |
  +-----------------------------+
                                |
                         special-app
                                |
                       +--------+--------+
                       |                 |
                 nodeSelector       toleration
                    matches           matches
```

This is a very useful production-style pattern.

---

# 27. Removing the Taint

When you're finished with the lab, remove the taint.

The syntax is:

```bash
kubectl taint node <NODE1> dedicated=special:NoSchedule-
```

Notice the **minus (`-`) at the end**.

Verify:

```bash
kubectl describe node <NODE1>
```

---

# 28. Remove the Label

You can also remove the label:

```bash
kubectl label node <NODE1> workload-
```

Verify:

```bash
kubectl get node <NODE1> --show-labels
```

---

# 29. Important Scheduling Flow

Finally, students should understand that the **scheduler is responsible for making the placement decision**.

Conceptually:

```text
                    Pod Created
                         |
                         v
                 Kubernetes API
                         |
                         v
                    Scheduler
                         |
              +----------+----------+
              |                     |
         Scheduling Rules       Node Resources
              |                     |
       +------+------+          +----+----+
       |      |      |          |         |
      Name  Selector Affinity   CPU      Memory
       |      |      |          |         |
       +------+------+----------+---------+
                         |
                         v
                  Suitable Node
                         |
                         v
                      Kubelet
                         |
                         v
                       Pod
```

The four mechanisms can then be remembered as:

```text
nodeName
   ↓
Exact node

nodeSelector
   ↓
Node label

Node Affinity
   ↓
Advanced node rules

Taint + Toleration
   ↓
Node restrictions
```

### One especially important distinction

```text
nodeSelector / Affinity
        |
        v
"What node should I select?"

Taint / Toleration
        |
        v
"Which Pods are allowed on this node?"
```

That distinction will make the later topics—**Deployments, multiple node pools, dedicated workloads, GPU nodes, system/user pools, and production scheduling**—much easier to understand.
