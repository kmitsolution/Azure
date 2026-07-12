# Learn Azure Container Apps Through Azure Portal (Step-by-Step Labs)

The best way to learn Azure Container Apps is:

```text id="n3z8th"
Portal → Understand Concepts
CLI → Automate
Terraform/Bicep → Infrastructure as Code
```

---

# Learning Path

```text id="0n7iwj"
1. Create Resource Group
2. Create Log Analytics Workspace
3. Create Container App Environment
4. Deploy First App
5. Configure Ingress
6. Configure Scaling
7. Configure Revisions
8. Configure Secrets
9. Configure Managed Identity
10. Configure Networking
11. Monitor Logs
12. Deploy Production Application
```

---

# Lab 1 – Create Resource Group

Go to:

```text id="d2olq0"
Azure Portal
     ↓
Resource Groups
     ↓
Create
```

Fill:

| Setting        | Value                |
| -------------- | -------------------- |
| Subscription   | Your Subscription    |
| Resource Group | rg-containerapps-dev |
| Region         | Central India        |

Click:

```text id="12fhj1"
Review + Create
```

---

# Lab 2 – Create Log Analytics Workspace

Container Apps requires Log Analytics.

Go to:

```text id="akqcr3"
Portal
   ↓
Log Analytics Workspace
   ↓
Create
```

Fill:

| Setting        | Value                |
| -------------- | -------------------- |
| Name           | law-containerapps    |
| Resource Group | rg-containerapps-dev |
| Region         | Central India        |

Click:

```text id="7t2m44"
Create
```

---

# Lab 3 – Create Container App Environment

This is one of the most important concepts.

Go to:

```text id="2ghjz0"
Portal
     ↓
Container App Environments
     ↓
Create
```

Fill:

### Basics

| Setting          | Value                |
| ---------------- | -------------------- |
| Environment Name | cae-dev              |
| Resource Group   | rg-containerapps-dev |
| Region           | Central India        |

---

### Monitoring

Select:

```text id="fxq1t5"
law-containerapps
```

---

### Networking (Leave default initially)

```text id="6txhsl"
Public Access
```

Click:

```text id="30k7pc"
Create
```

---

# Understand Environment Architecture

```text id="4bbl5o"
Container App Environment
      │
      ├── App1
      ├── App2
      ├── App3
      └── Internal DNS
```

Think of this as:

```text id="ktg8xe"
Managed Kubernetes Cluster
```

---

# Lab 4 – Deploy First Container App

Go to:

```text id="b6p3y9"
Container Apps
      ↓
Create
```

---

## Basics Tab

| Setting        | Value                |
| -------------- | -------------------- |
| Name           | app-nginx            |
| Resource Group | rg-containerapps-dev |
| Environment    | cae-dev              |
| Region         | Central India        |

---

## Container Tab

Container Image:

```text id="zbj8if"
nginx
```

CPU:

```text id="1g4z7t"
0.5
```

Memory:

```text id="lk6jyy"
1 GB
```

---

## Ingress Tab

Enable:

```text id="5ul7al"
Ingress = Enabled
```

Type:

```text id="o6u18x"
External
```

Target Port:

```text id="mbolaf"
80
```

Click:

```text id="f79t0l"
Review + Create
```

After deployment you will get:

```text id="2vzwtv"
https://xxxxx.centralindia.azurecontainerapps.io
```

Open URL.

You should see:

```text id="ngxy1x"
Welcome to nginx!
```

---

# Understand What Got Created

```text id="fg9qeb"
Resource Group
     │
     ├── Container App Environment
     ├── Container App
     ├── Log Analytics
     └── Managed Resources
```

---

# Lab 5 – Explore Container App Blade

Open:

```text id="jyrb7t"
Container Apps
      ↓
app-nginx
```

Observe:

### Overview

Contains:

* URL
* Status
* Environment
* Replicas
* Revisions

---

# Lab 6 – Understand Revisions

Go to:

```text id="q0m5sh"
app-nginx
     ↓
Revisions
```

You will see:

```text id="yq0fcd"
Revision Name
Traffic %
Status
Replicas
```

Think of revision as:

```text id="a0e8w2"
Application Version
```

---

## Create New Revision

Go to:

```text id="mujw31"
Containers
```

Change image:

```text id="swpq8u"
nginx:latest
```

Save.

A new revision gets created automatically.

---

# Lab 7 – Traffic Splitting

Go to:

```text id="o4hbo5"
Revisions
      ↓
Multiple Revision Mode
```

Example:

```text id="v0rtto"
Revision1 → 80%
Revision2 → 20%
```

This is useful for:

* Canary Deployment
* Blue Green Deployment

---

# Lab 8 – Scaling

Go to:

```text id="ux24p8"
Scale
```

Set:

| Setting      | Value |
| ------------ | ----- |
| Min Replicas | 0     |
| Max Replicas | 10    |

---

## HTTP Scaling Rule

Add rule:

```text id="hyh53c"
Concurrent Requests = 50
```

Architecture:

```text id="2f8ie9"
Users
   ↓
Requests Increase
   ↓
Scale Out
```

---

# Lab 9 – Scale to Zero

Set:

```text id="cmf6vx"
Minimum Replicas = 0
```

Wait.

You will notice:

```text id="2klngw"
Replicas = 0
```

This is one of the biggest advantages over AKS.

---

# Lab 10 – Environment Variables

Go to:

```text id="nnj0im"
Containers
      ↓
Environment Variables
```

Add:

```text id="iyf4s4"
APP_ENV=DEV
```

Redeploy.

---

# Lab 11 – Secrets

Go to:

```text id="u0f6yc"
Secrets
```

Create:

```text id="0idb4e"
dbpassword
Password123
```

You can use this inside your container.

---

# Lab 12 – Managed Identity

Go to:

```text id="8m4jkh"
Identity
```

Enable:

```text id="7ysf9t"
System Assigned Identity
```

This allows access to:

* Key Vault
* Storage Account
* Service Bus

without storing credentials.

---

# Lab 13 – Networking

Go to:

```text id="r2owv4"
Networking
```

Learn:

### External Ingress

```text id="8i77j7"
Internet
      ↓
Container App
```

### Internal Ingress

```text id="y9jk34"
Container App
      ↓
Container App
```

---

# Lab 14 – Logs

Go to:

```text id="tps5q6"
Monitoring
      ↓
Logs
```

Example query:

```kql id="r32j7l"
ContainerAppConsoleLogs_CL
| order by TimeGenerated desc
```

Useful queries:

### Errors

```kql id="w8fr5g"
ContainerAppConsoleLogs_CL
| where Log_s contains "error"
```

### Last 50 Logs

```kql id="gzwxrw"
ContainerAppConsoleLogs_CL
| take 50
```

---

# Lab 15 – Metrics

Go to:

```text id="1mtb8w"
Monitoring
      ↓
Metrics
```

Observe:

* Requests
* CPU
* Memory
* Replica Count

---

# Lab 16 – Replica Explorer

Go to:

```text id="zsqb20"
Replicas
```

Observe:

```text id="zv1e6v"
Replica1
Replica2
Replica3
```

Equivalent to:

```text id="57ag1o"
Pods in Kubernetes
```

---

# Lab 17 – Custom Domain

Go to:

```text id="uk3x2w"
Custom Domains
```

Map:

```text id="ol30kj"
api.mycompany.com
```

Azure automatically manages certificates.

---

# Lab 18 – Deploy from Azure Container Registry

Create ACR:

```text id="7l4m6t"
myacr123
```

Push image.

Go to:

```text id="wskfo3"
Containers
      ↓
Image Source
```

Select:

```text id="mztjzv"
Azure Container Registry
```

Deploy.

---

# Lab 19 – Create Internal Microservices

Create:

```text id="yljlwm"
frontend-api
payment-api
notification-api
```

Set:

```text id="4sl8nq"
Ingress = Internal
```

Communication:

```text id="j49bf6"
frontend-api
      ↓
payment-api
```

This is where ACA becomes very powerful.

---

# Lab 20 – Container App Jobs

Go to:

```text id="vuzjln"
Container App Jobs
```

Examples:

* Backup Jobs
* ETL Jobs
* Report Generation

Equivalent to:

```text id="b40d6i"
Kubernetes CronJobs
```

---

# Suggested Practice Project

## Banking Application

```text id="5jyy5q"
Container App Environment
│
├── customer-api
├── account-api
├── payment-api
├── notification-api
└── worker-job
```

Add:

* Scaling
* Secrets
* Managed Identity
* Logging
* Revisions

---

# Recommended Portal Learning Sequence

```text id="2v6u6e"
Portal Basics
      ↓
Environment
      ↓
Deploy nginx
      ↓
Ingress
      ↓
Scaling
      ↓
Revisions
      ↓
Secrets
      ↓
Identity
      ↓
Networking
      ↓
Logs
      ↓
Jobs
      ↓
Microservices
      ↓
CI/CD
```

---

# Portal Mapping with AKS Concepts

| Azure Container Apps | AKS Equivalent       |
| -------------------- | -------------------- |
| Environment          | Cluster              |
| Container App        | Deployment + Service |
| Replica              | Pod                  |
| Revision             | ReplicaSet           |
| Scale Rules          | HPA/KEDA             |
| Jobs                 | CronJobs             |
| Internal DNS         | Kubernetes Service   |

The biggest learning objective in the Portal should be understanding:

```text id="k4kfnw"
Environment
Revisions
Replicas
Ingress
Scaling
Jobs
```

because these concepts directly translate to Kubernetes and AKS later.
