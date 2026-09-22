# AKS Cluster Health Monitoring, Capacity Planning & Resource Optimization

These are three closely related topics after **AKS Monitoring and Alerting**:

```text
AKS Monitoring
      ↓
Cluster Health Monitoring
      ↓
Capacity Planning
      ↓
Resource Optimization
```

The goal is to answer three questions:

1. **Is my AKS cluster healthy?**
2. **Will the cluster have enough capacity as the workload grows?**
3. **Am I wasting CPU, memory, or nodes?**

---

# 1. Cluster Health Monitoring

Cluster health monitoring means continuously checking whether the AKS cluster and its workloads are operating normally.

We monitor:

```text
AKS Cluster
│
├── Nodes
│   ├── Ready / NotReady
│   ├── CPU
│   ├── Memory
│   ├── Disk
│   └── Network
│
├── Pods
│   ├── Running
│   ├── Pending
│   ├── Failed
│   ├── Restarting
│   └── CrashLoopBackOff
│
├── Deployments
│   ├── Desired replicas
│   └── Available replicas
│
└── Kubernetes events
    ├── Scheduling problems
    ├── Image pull errors
    └── Container failures
```

---

# 2. First Check — Cluster Nodes

Use:

```bash
kubectl get nodes
```

Example:

```text
NAME                                STATUS   ROLES
aks-nodepool1-12345678-vmss000000   Ready    <none>
aks-nodepool1-12345678-vmss000001   Ready    <none>
aks-nodepool1-12345678-vmss000002   Ready    <none>
```

A healthy cluster should normally have the expected nodes in:

```text
Ready
```

---

# 3. Detailed Node Information

```bash
kubectl describe nodes
```

Or for one node:

```bash
kubectl describe node aks-nodepool1-12345678-vmss000000
```

Look particularly at:

```text
Conditions
Capacity
Allocatable
Allocated resources
Events
```

---

# 4. Check Node Conditions

```bash
kubectl get nodes
```

For more detail:

```bash
kubectl get nodes -o wide
```

You can also check conditions:

```bash
kubectl get nodes \
  -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[-1].type
```

More importantly, `kubectl describe node` shows conditions such as:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
```

For example:

```text
Ready          True
MemoryPressure False
DiskPressure   False
PIDPressure    False
```

This is a useful basic health check.

---

# 5. Check Pods

```bash
kubectl get pods -A
```

Example:

```text
NAMESPACE     NAME                         READY   STATUS
default       nginx-7c79c4bf97-x8abc       1/1     Running
kube-system   coredns-xxxx                 1/1     Running
kube-system   kube-proxy-xxxxx             1/1     Running
```

Look for:

```text
Running
Pending
Failed
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
```

---

# 6. Check Restarting Pods

```bash
kubectl get pods -A
```

Pay attention to:

```text
RESTARTS
```

Example:

```text
NAME                    READY   STATUS    RESTARTS
nginx-xxx               1/1     Running   0
api-xxx                 1/1     Running   12
```

A high or continuously increasing restart count requires investigation.

---

# 7. Check Deployments

```bash
kubectl get deployments -A
```

Example:

```text
NAMESPACE   NAME     READY   UP-TO-DATE   AVAILABLE
default     nginx    3/3     3            3
```

Healthy:

```text
READY       3/3
AVAILABLE   3
```

Potential problem:

```text
READY       2/3
AVAILABLE   2
```

This means the deployment wants three replicas but only two are currently available.

---

# 8. Check Cluster Events

One of the simplest troubleshooting commands:

```bash
kubectl get events -A \
  --sort-by=.lastTimestamp
```

You may see:

```text
FailedScheduling
Failed
BackOff
Pulling
Pulled
Created
Started
```

For example:

```text
Warning   FailedScheduling
0/3 nodes are available
```

This immediately tells us there is a scheduling/capacity problem.

---

# 9. Resource Usage

Now we move from **health** to **resource utilization**.

Use:

```bash
kubectl top nodes
```

Example:

```text
NAME          CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node-000      850m         42%    3200Mi          41%
node-001      1200m        60%    4100Mi          52%
node-002      600m         30%    3000Mi          38%
```

This gives us a quick picture of node utilization.

---

# 10. Pod Resource Usage

```bash
kubectl top pods -A
```

Example:

```text
NAMESPACE   NAME        CPU(cores)   MEMORY
default     nginx       20m          30Mi
default     api         450m         700Mi
default     worker      800m         1Gi
```

Now we can identify resource-heavy workloads.

---

# 11. Azure Monitor View

The same concept can be viewed through:

```text
Azure Portal
    ↓
AKS
    ↓
Monitoring
    ↓
Insights
```

You can investigate:

```text
Nodes
Controllers
Containers
```

This is useful because Azure Monitor gives you historical information instead of only the current snapshot.

---

# 12. Cluster Health Checklist

For a simple health check, use:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get deployments -A
kubectl get events -A
kubectl top nodes
kubectl top pods -A
```

Think:

```text
Nodes       → Are nodes healthy?
Pods        → Are applications healthy?
Deployment  → Are replicas available?
Events      → Are there errors?
Metrics     → Are resources under pressure?
```

---

# 13. Capacity Planning

Now suppose your application is growing.

Today:

```text
10 Pods
```

Next month:

```text
30 Pods
```

Next year:

```text
100 Pods
```

We need to determine:

> How much AKS capacity will we need?

This is **capacity planning**.

---

# 14. Example

Suppose we have:

```text
3 Nodes
```

Each node:

```text
4 vCPU
16 GB RAM
```

Total theoretical capacity:

```text
CPU:
3 × 4 = 12 vCPU

Memory:
3 × 16 = 48 GB
```

But we should not treat the entire amount as available application capacity.

Kubernetes and system components also consume resources.

Therefore:

```text
Physical capacity
        ↓
System reservations
        ↓
Available capacity
        ↓
Application workloads
```

---

# 15. Requests vs Limits

This is one of the most important concepts for capacity planning.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
```

### Request

The request tells Kubernetes:

> I need at least this much resource for scheduling.

### Limit

The limit tells Kubernetes:

> Don't allow this container to use more than this amount.

---

# 16. Capacity Planning Example

Suppose we have:

```text
20 Pods
```

Each pod requests:

```text
CPU: 250m
Memory: 512Mi
```

Total CPU request:

```text
20 × 250m = 5000m
```

Therefore:

```text
5000m = 5 CPU cores
```

Memory:

```text
20 × 512Mi
= 10240Mi
≈ 10 GiB
```

So our workloads require approximately:

```text
5 CPU cores
10 GiB memory
```

plus system overhead and appropriate headroom.

---

# 17. Why Requests Matter for Capacity

Suppose a node has:

```text
4 CPU
```

And existing pods have requests:

```text
3.5 CPU
```

There may only be:

```text
0.5 CPU
```

available for scheduling additional workloads, even if current actual CPU utilization looks lower.

This is an important distinction:

```text
Actual usage ≠ Scheduling capacity
```

Kubernetes schedules pods based primarily on **resource requests**, not simply current CPU utilization.

---

# 18. Pending Pods and Capacity

Consider:

```bash
kubectl get pods
```

You see:

```text
api-xxxxx    Pending
```

Check:

```bash
kubectl describe pod api-xxxxx
```

At the bottom, you may find:

```text
Warning  FailedScheduling

0/3 nodes are available:
Insufficient cpu
```

This is a capacity problem.

---

# 19. Capacity Planning Strategy

A simple process:

```text
1. Measure current usage
        ↓
2. Measure resource requests
        ↓
3. Understand workload growth
        ↓
4. Calculate required capacity
        ↓
5. Keep headroom
        ↓
6. Configure autoscaling
        ↓
7. Monitor continuously
```

---

# 20. Cluster Autoscaler

If workloads increase and pods cannot be scheduled, AKS can use the **Cluster Autoscaler** to add nodes to the node pool.

Conceptually:

```text
              AKS
               │
          Node Pool
        ┌──────┼──────┐
       Node   Node   Node
        │
        ▼
     New Pods
        │
        ▼
Insufficient capacity
        │
        ▼
Cluster Autoscaler
        │
        ▼
     New Node
```

---

# 21. Horizontal Pod Autoscaler

HPA operates at the **pod/application level**.

Example:

```text
Low traffic
    ↓
2 Pods

High traffic
    ↓
5 Pods
```

Command:

```bash
kubectl autoscale deployment nginx \
  --cpu-percent=70 \
  --min=2 \
  --max=10
```

Now Kubernetes can increase or decrease the number of replicas based on the configured metric.

---

# 22. Cluster Autoscaler vs HPA

Very important:

| Feature | HPA                     | Cluster Autoscaler          |
| ------- | ----------------------- | --------------------------- |
| Scales  | Pods                    | Nodes                       |
| Level   | Application             | Infrastructure              |
| Example | 2 → 10 pods             | 3 → 5 nodes                 |
| Trigger | Resource/custom metrics | Unschedulable pods/capacity |
| Purpose | Application scaling     | Cluster capacity            |

They can work together:

```text
Traffic increases
       ↓
HPA increases Pods
       ↓
Pods cannot fit
       ↓
Cluster Autoscaler
       ↓
Adds Nodes
       ↓
Pods get scheduled
```

---

# 23. Resource Optimization

Capacity planning asks:

> How much capacity do I need?

Resource optimization asks:

> Am I using that capacity efficiently?

---

# 24. Example of Poor Resource Configuration

Suppose we configure:

```yaml
resources:
  requests:
    cpu: "2"
    memory: "4Gi"

  limits:
    cpu: "4"
    memory: "8Gi"
```

But the application normally uses:

```text
CPU: 200m
Memory: 500Mi
```

We have significantly overestimated the request.

If many pods have unnecessarily large requests:

```text
Node capacity
      ↓
Large requests
      ↓
Fewer pods can be scheduled
      ↓
Need more nodes
      ↓
Higher cost
```

---

# 25. Right-Sizing

Resource optimization involves **right-sizing** requests and limits based on observed usage.

For example:

Current:

```text
CPU request:    2 CPU
Actual usage:   200m
```

After analysis, we might determine that a lower request is appropriate, with enough headroom for normal workload variation.

The exact value should come from observed workload behavior rather than an arbitrary number.

---

# 26. CPU Optimization

Check:

```bash
kubectl top pods -A
```

Suppose:

```text
api     50m
worker  900m
nginx   10m
```

The worker is consuming substantially more CPU.

Investigate:

```text
Is the workload expected?
Is the request appropriate?
Is there a performance problem?
Can the application be optimized?
Should HPA be used?
```

---

# 27. Memory Optimization

Memory is especially important because memory exhaustion can result in OOM kills.

Example:

```text
Memory request: 2Gi
Actual usage:   300Mi
```

This may indicate an opportunity for right-sizing.

But don't immediately reduce it.

Check:

```text
Normal usage
Peak usage
Traffic spikes
Application behavior
OOM events
```

Then choose an appropriate value.

---

# 28. Requests and Limits Strategy

A practical approach:

```text
Measure
  ↓
Observe normal usage
  ↓
Observe peak usage
  ↓
Set request around expected scheduling requirement
  ↓
Set limit according to application behavior
  ↓
Monitor
  ↓
Adjust
```

This is a continuous optimization process.

---

# 29. Resource Quotas

For multi-team AKS clusters, one team shouldn't consume all available resources.

Kubernetes supports:

**ResourceQuota**

Example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-quota
  namespace: development
spec:
  requests.cpu: "4"
  requests.memory: 8Gi
  limits.cpu: "8"
  limits.memory: 16Gi
```

Apply:

```bash
kubectl apply -f quota.yaml
```

Check:

```bash
kubectl get resourcequota -n development
```

This helps control resource consumption at the namespace level.

---

# 30. LimitRange

`LimitRange` can provide default requests and limits.

Example:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - type: Container
    default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
```

This prevents developers from creating containers without reasonable resource settings.

---

# 31. Three Concepts Together

The relationship is:

```text
             AKS Monitoring
                   │
        ┌──────────┴──────────┐
        │                     │
   Cluster Health       Resource Metrics
        │                     │
        │                     ▼
        │              Capacity Planning
        │                     │
        │                     ▼
        │                Autoscaling
        │                     │
        └──────────────┬──────┘
                       ▼
                Resource Optimization
```

---

# 32. Practical AKS Example

Suppose we have:

```text
AKS
│
├── 3 Nodes
│
├── nginx       2 replicas
├── frontend    3 replicas
├── backend     3 replicas
└── worker      2 replicas
```

Monitoring shows:

```text
CPU:    75%
Memory: 80%
```

### Health monitoring

Check:

```bash
kubectl get nodes
kubectl get pods -A
```

Everything is `Running`.

### Capacity planning

Traffic is increasing.

HPA may increase:

```text
Backend:
3 → 8 replicas
```

If nodes don't have enough capacity:

```text
Pending pods
      ↓
Cluster Autoscaler
      ↓
3 nodes → 5 nodes
```

### Resource optimization

After several days we discover:

```text
frontend request:
500m CPU

actual:
100m CPU
```

We can review and right-size the request.

---

# 33. Production Monitoring Strategy

A practical AKS strategy can look like this:

```text
                    AKS
                     │
          ┌──────────┼──────────┐
          │          │          │
        Health     Metrics     Logs
          │          │          │
          ▼          ▼          ▼
       Nodes       CPU/RAM    Errors
       Pods        Network    Events
       Events      Requests   Logs
          │          │          │
          └──────────┼──────────┘
                     ▼
               Azure Monitor
                     │
            ┌────────┴────────┐
            │                 │
          Alerts          Analysis
            │                 │
            ▼                 ▼
       Action Groups      Capacity
                            │
                            ▼
                       Optimization
```

---

# 34. Useful Commands Cheat Sheet

### Cluster health

```bash
kubectl get nodes
kubectl get nodes -o wide
kubectl get pods -A
kubectl get deployments -A
kubectl get events -A --sort-by=.lastTimestamp
```

### Resource usage

```bash
kubectl top nodes
kubectl top pods -A
```

### Resource configuration

```bash
kubectl describe node <node-name>
kubectl describe pod <pod-name>
```

### Autoscaling

```bash
kubectl get hpa -A
```

### Quotas

```bash
kubectl get resourcequota -A
kubectl get limitrange -A
```

---

# 35. Key Points to Remember

### Cluster Health Monitoring

Focus on:

```text
Nodes
Pods
Deployments
Events
Restarts
CPU
Memory
```

### Capacity Planning

Focus on:

```text
Resource requests
Available capacity
Workload growth
HPA
Cluster Autoscaler
Headroom
```

### Resource Optimization

Focus on:

```text
Right-sizing
CPU utilization
Memory utilization
Requests/limits
ResourceQuota
LimitRange
Autoscaling
```

The overall objective is:

> **Keep the AKS cluster healthy, have enough capacity for workload growth, and avoid paying for resources that aren't needed.**
