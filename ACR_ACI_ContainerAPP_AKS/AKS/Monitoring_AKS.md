# AKS Monitoring with Azure Monitor — Complete Practical Example

For AKS, **Azure Monitor** can collect both **logs** and **metrics**. A simple learning setup is:

* **Container Insights** → container/pod/node logs and inventory
* **Log Analytics Workspace** → stores/query logs using KQL
* **Azure Monitor managed service for Prometheus** → Kubernetes metrics
* **Azure Managed Grafana** → visualize Prometheus metrics

Microsoft currently recommends these as complementary monitoring capabilities for AKS. ([Microsoft Learn][1])

---

## 1. Architecture

```text
                    AKS Cluster
                 ┌───────────────┐
                 │               │
                 │  Node         │
                 │   ├─ Pod      │
                 │   ├─ Pod      │
                 │   └─ Pod      │
                 │               │
                 └───────┬───────┘
                         │
             Azure Monitor Agent
                    /          \
                   /            \
                  ▼              ▼
        Container Insights    Prometheus
                  │              │
                  ▼              ▼
          Log Analytics      Azure Monitor
             Workspace          Workspace
                  │              │
                  ▼              ▼
              KQL Query      Grafana
```

Think of it this way:

| Component               | Purpose                                   |
| ----------------------- | ----------------------------------------- |
| Container Insights      | Collects AKS/container logs and inventory |
| Log Analytics           | Stores logs                               |
| KQL                     | Queries logs                              |
| Managed Prometheus      | Collects Kubernetes metrics               |
| Azure Monitor Workspace | Stores Prometheus metrics                 |
| Managed Grafana         | Displays dashboards                       |

Container Insights uses Azure Monitor Agent and stores collected AKS/container data in Log Analytics. ([Microsoft Learn][1])

---

# 2. Example AKS Cluster

Let's assume:

```powershell
$RG = "MYRG-India"
$AKS = "myaks"
```

Check your cluster:

### PowerShell

```powershell
az aks show `
  --name $AKS `
  --resource-group $RG `
  --output table
```

### Bash

```bash
az aks show \
  --name $AKS \
  --resource-group $RG \
  --output table
```

---

# 3. Enable Container Insights

Container Insights is the easiest starting point because it allows us to monitor:

* Nodes
* Pods
* Containers
* CPU
* Memory
* Kubernetes events
* Container logs
* Inventory

Microsoft supports enabling it on an existing AKS cluster using `az aks enable-addons --addon monitoring`. ([Microsoft Learn][1])

## Option 1 — Let Azure use/create the default Log Analytics Workspace

### PowerShell

```powershell
az aks enable-addons `
  --addon monitoring `
  --name $AKS `
  --resource-group $RG
```

### Bash

```bash
az aks enable-addons \
  --addon monitoring \
  --name $AKS \
  --resource-group $RG
```

---

# 4. Verify Monitoring

Run:

### PowerShell / Bash

```bash
az aks show \
  --name $AKS \
  --resource-group $RG \
  --query addonProfiles.omsagent.enabled \
  --output tsv
```

Expected:

```text
true
```

You can also check:

```bash
az aks show \
  --name $AKS \
  --resource-group $RG \
  --query addonProfiles
```

---

# 5. Create a Simple Application

Let's deploy NGINX.

```bash
kubectl create deployment nginx \
  --image=nginx
```

Check:

```bash
kubectl get pods
```

Example:

```text
NAME                     READY   STATUS    RESTARTS
nginx-7c79c4bf97-x8abc   1/1     Running   0
```

Create a service:

```bash
kubectl expose deployment nginx \
  --port=80 \
  --type=ClusterIP
```

---

# 6. Generate Some Logs

Monitoring becomes easier to demonstrate if the application generates logs.

Get the pod:

```bash
kubectl get pods
```

Then generate requests:

```bash
kubectl exec deployment/nginx -- curl localhost
```

Run it several times:

```bash
kubectl exec deployment/nginx -- curl localhost
kubectl exec deployment/nginx -- curl localhost
kubectl exec deployment/nginx -- curl localhost
```

NGINX will generate access-log entries.

---

# 7. View Monitoring in Azure Portal

Go to:

**Azure Portal → Kubernetes services → your AKS cluster**

Then:

**Monitoring → Insights**

You should see information about:

### Nodes

```text
Node
CPU
Memory
Disk
Network
```

### Controllers

```text
Deployment
ReplicaSet
DaemonSet
StatefulSet
```

### Containers

```text
Container
CPU
Memory
Restart count
```

### Pods

```text
Pod
Status
CPU
Memory
Restarts
```

Microsoft documents AKS Insights as the portal experience for viewing monitoring information from the cluster. ([Microsoft Learn][2])

---

# 8. Query Container Logs Using KQL

This is one of the most important parts to demonstrate.

Go to:

**Azure Portal → Log Analytics Workspace → Logs**

Depending on your monitoring configuration, container logs can appear in tables such as `ContainerLogV2`. Microsoft recommends `ContainerLogV2` for container logging when using the current monitoring configuration. ([Microsoft Learn][1])

Try:

```kusto
ContainerLogV2
| take 20
```

---

# 9. Find NGINX Logs

```kusto
ContainerLogV2
| where PodName contains "nginx"
| project TimeGenerated, PodName, ContainerName, LogMessage
| order by TimeGenerated desc
```

You should get something similar to:

```text
TimeGenerated             PodName                 LogMessage
------------------------------------------------------------
10:25:31                  nginx-7c79...           10.0.0.5 - GET /
10:25:30                  nginx-7c79...           10.0.0.5 - GET /
10:25:28                  nginx-7c79...           10.0.0.5 - GET /
```

This demonstrates:

```text
AKS
 ↓
Azure Monitor Agent
 ↓
Container Insights
 ↓
Log Analytics
 ↓
KQL
```

---

# 10. Query Kubernetes Events

You can also query Kubernetes events.

```kusto
KubeEvents
| order by TimeGenerated desc
| take 20
```

This can help identify things such as:

```text
Pod scheduled
Container started
Container failed
Image pull error
Pod restarted
```

---

# 11. Monitor CPU and Memory

Container Insights also collects performance information.

For example:

```kusto
InsightsMetrics
| where Namespace == "container.azm.ms"
| take 20
```

You can use this to investigate CPU, memory and other performance information.

---

# 12. Check Nodes

In the Azure Portal:

**AKS → Insights → Nodes**

You can see information such as:

```text
Node Name
CPU Usage
Memory Usage
Disk
Network
```

This is useful for answering questions like:

> Is my AKS node running out of CPU?

---

# 13. Check Pods

Go to:

**AKS → Insights → Controllers**

or the appropriate workload/pod views.

You can investigate:

```text
Pod status
CPU
Memory
Restarts
Container status
```

For example:

```text
nginx-7c79c4bf97-x8abc
CPU       2%
Memory    35Mi
Restarts  0
```

---

# 14. Now Add Managed Prometheus

Container Insights and Prometheus have different purposes.

### Container Insights

Primarily useful for:

```text
Logs
Container inventory
Pod information
Node information
Kubernetes events
```

### Prometheus

Primarily useful for:

```text
Metrics
Kubernetes metrics
Application metrics
CPU
Memory
Request rate
Error rate
Custom metrics
```

Azure Monitor's managed Prometheus service sends Prometheus metrics to an **Azure Monitor workspace**, which can then be visualized with Azure Managed Grafana. ([Microsoft Learn][3])

---

# 15. Create Azure Monitor Workspace

Example:

### PowerShell

```powershell
$MONITOR = "myaks-monitor"
```

Then:

```powershell
az resource create `
  --resource-group $RG `
  --namespace microsoft.monitor `
  --resource-type accounts `
  --name $MONITOR `
  --location centralindia `
  --properties "{}"
```

### Bash

```bash
MONITOR="myaks-monitor"

az resource create \
  --resource-group $RG \
  --namespace microsoft.monitor \
  --resource-type accounts \
  --name $MONITOR \
  --location centralindia \
  --properties '{}'
```

Microsoft's current AKS monitoring documentation uses an Azure Monitor workspace for managed Prometheus metrics. ([Microsoft Learn][4])

---

# 16. Get Azure Monitor Workspace ID

### PowerShell

```powershell
$MONITOR_ID = az resource show `
  --resource-group $RG `
  --name $MONITOR `
  --resource-type "Microsoft.Monitor/accounts" `
  --query id `
  --output tsv

$MONITOR_ID
```

### Bash

```bash
MONITOR_ID=$(az resource show \
  --resource-group $RG \
  --name $MONITOR \
  --resource-type "Microsoft.Monitor/accounts" \
  --query id \
  --output tsv)

echo $MONITOR_ID
```

---

# 17. Enable Azure Monitor Metrics

For an existing AKS cluster:

### PowerShell

```powershell
az aks update `
  --name $AKS `
  --resource-group $RG `
  --enable-azure-monitor-metrics `
  --azure-monitor-workspace-resource-id $MONITOR_ID
```

### Bash

```bash
az aks update \
  --name $AKS \
  --resource-group $RG \
  --enable-azure-monitor-metrics \
  --azure-monitor-workspace-resource-id $MONITOR_ID
```

The current CLI supports `--enable-azure-monitor-metrics` and an Azure Monitor workspace resource ID. ([Microsoft Learn][5])

---

# 18. Managed Grafana

Grafana gives us dashboards.

Create it:

### PowerShell

```powershell
$GRAFANA = "myaks-grafana"

az grafana create `
  --name $GRAFANA `
  --resource-group $RG
```

### Bash

```bash
GRAFANA="myaks-grafana"

az grafana create \
  --name $GRAFANA \
  --resource-group $RG
```

Microsoft documents Azure Managed Grafana as the visualization layer for Azure Monitor managed Prometheus. ([Microsoft Learn][4])

---

# 19. Get Grafana Resource ID

### PowerShell

```powershell
$GRAFANA_ID = az grafana show `
  --name $GRAFANA `
  --resource-group $RG `
  --query id `
  --output tsv

$GRAFANA_ID
```

### Bash

```bash
GRAFANA_ID=$(az grafana show \
  --name $GRAFANA \
  --resource-group $RG \
  --query id \
  --output tsv)

echo $GRAFANA_ID
```

---

# 20. Connect AKS → Azure Monitor → Grafana

### PowerShell

```powershell
az aks update `
  --name $AKS `
  --resource-group $RG `
  --enable-azure-monitor-metrics `
  --azure-monitor-workspace-resource-id $MONITOR_ID `
  --grafana-resource-id $GRAFANA_ID
```

### Bash

```bash
az aks update \
  --name $AKS \
  --resource-group $RG \
  --enable-azure-monitor-metrics \
  --azure-monitor-workspace-resource-id $MONITOR_ID \
  --grafana-resource-id $GRAFANA_ID
```

This establishes the relationship:

```text
AKS
 │
 │ Prometheus metrics
 ▼
Azure Monitor Workspace
 │
 ▼
Azure Managed Grafana
 │
 ▼
Dashboards
```

---

# 21. Very Simple Demo

For your training, I would demonstrate it in this order:

### Step 1 — Create AKS

```text
AKS Cluster
```

### Step 2 — Deploy NGINX

```bash
kubectl create deployment nginx --image=nginx
```

### Step 3 — Generate traffic

```bash
kubectl exec deployment/nginx -- curl localhost
```

### Step 4 — Enable Container Insights

```bash
az aks enable-addons \
  --addon monitoring \
  --name $AKS \
  --resource-group $RG
```

### Step 5 — Open

```text
AKS
 ↓
Monitoring
 ↓
Insights
```

Show:

```text
Nodes
Controllers
Containers
```

### Step 6 — Query logs

```kusto
ContainerLogV2
| where PodName contains "nginx"
| order by TimeGenerated desc
```

### Step 7 — Enable Managed Prometheus

```bash
az aks update \
  --name $AKS \
  --resource-group $RG \
  --enable-azure-monitor-metrics \
  --azure-monitor-workspace-resource-id $MONITOR_ID
```

### Step 8 — Connect Grafana

```text
AKS
 ↓
Prometheus
 ↓
Azure Monitor Workspace
 ↓
Azure Managed Grafana
```

---

# 22. Important Difference

This is worth emphasizing in your AZ-104/AKS training:

```text
                    AKS
                     │
          ┌──────────┴──────────┐
          │                     │
       LOGS                   METRICS
          │                     │
          ▼                     ▼
 Container Insights      Managed Prometheus
          │                     │
          ▼                     ▼
 Log Analytics          Azure Monitor Workspace
          │                     │
          ▼                     ▼
          KQL                 Grafana
```

**Log Analytics ≠ Azure Monitor Workspace.**

For the common AKS monitoring architecture:

* **Log Analytics Workspace** → container logs and KQL
* **Azure Monitor Workspace** → managed Prometheus metrics
* **Grafana** → visualization of metrics

That distinction is particularly useful when explaining Azure Monitor to students. ([Microsoft Learn][3])

### Current Microsoft documentation

[Enable monitoring for AKS clusters](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/kubernetes-monitoring-enable?source=post_page---------------------------&utm_source=chatgpt.com)
[Monitor AKS](https://learn.microsoft.com/en-us/azure/aks/monitor-aks?utm_source=chatgpt.com)
[Azure CLI — AKS commands](https://learn.microsoft.com/en-us/cli/azure/aks?++view=azure-cli-latest&view=azure-cli-latest&utm_source=chatgpt.com)

[1]: https://learn.microsoft.com/en-us/azure/azure-monitor/containers/kubernetes-monitoring-enable?source=post_page---------------------------&utm_source=chatgpt.com "Enable Monitoring for Azure Kubernetes Service (AKS) Clusters - Azure Monitor | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/azure/aks/kubernetes-portal?utm_source=chatgpt.com "Access Kubernetes Resources using the Azure Portal - Azure Kubernetes Service | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/azure/azure-monitor/containers/kubernetes-monitoring-tutorial?utm_source=chatgpt.com "Quickstart monitoring a Kubernetes cluster in Azure Monitor - Azure Monitor | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/azure/aks/advanced-network-observability-cli?tabs=non-cilium&utm_source=chatgpt.com "Set up Container Network Observability for Azure Kubernetes Service (AKS) - Azure managed Prometheus and Grafana - Azure Kubernetes Service | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/cli/azure/aks?++view=azure-cli-latest&view=azure-cli-latest&utm_source=chatgpt.com "az aks | Microsoft Learn"
