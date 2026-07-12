# Azure Container Apps Learning Roadmap (Portal + Azure CLI)

This roadmap assumes you already know Docker and basic Azure concepts.

---

# Phase 1 – Understand the Fundamentals (1 Day)

## Topics to Learn

### 1. What is Azure Container Apps?

* Serverless container platform
* Scale to zero
* Event-driven applications
* Microservices platform

### 2. Understand Architecture

```text
Container Image
      ↓
Container App
      ↓
Container App Environment
      ↓
Azure Infrastructure
```

### Learn Important Components

1. Container App Environment
2. Container App
3. Revision
4. Ingress
5. Scaling Rules
6. Dapr
7. Managed Identity
8. Secrets
9. Jobs

---

# Phase 2 – Setup Azure Environment (1 Day)

## Install Azure CLI

```bash
az version
```

If not installed:

[Azure CLI Installation Guide](https://learn.microsoft.com/cli/azure/install-azure-cli?utm_source=chatgpt.com)

---

## Login

```bash
az login
```

Select subscription:

```bash
az account set --subscription "<subscription-name>"
```

Verify:

```bash
az account show
```

---

## Install Container Apps Extension

```bash
az extension add --name containerapp
```

Update:

```bash
az extension update --name containerapp
```

Verify:

```bash
az containerapp --help
```

---

# Phase 3 – Learn Through Azure Portal (2 Days)

## Lab 1 – Create Resource Group

Portal:

```text
Resource Groups
     ↓
Create
```

Create:

```text
RG: rg-containerapp-dev
Region: Central India
```

CLI:

```bash
az group create \
--name rg-containerapp-dev \
--location centralindia
```

---

# Lab 2 – Create Container App Environment

Portal:

```text
Container App Environment
      ↓
Create
```

Understand:

* Log Analytics Workspace
* VNET Integration
* Zone Redundancy

CLI:

```bash
az monitor log-analytics workspace create \
-g rg-containerapp-dev \
-n law-containerapps
```

Create environment:

```bash
az containerapp env create \
-n cae-dev \
-g rg-containerapp-dev \
--logs-workspace-id <workspace-id> \
--logs-workspace-key <key> \
--location centralindia
```

---

# Lab 3 – Deploy First Application

Deploy Nginx.

Portal:

```text
Container Apps
      ↓
Create
```

Image:

```text
nginx
```

Ingress:

```text
External
Port:80
```

CLI:

```bash
az containerapp create \
-n app-nginx \
-g rg-containerapp-dev \
--environment cae-dev \
--image nginx \
--target-port 80 \
--ingress external
```

Check:

```bash
az containerapp show \
-n app-nginx \
-g rg-containerapp-dev
```

Open application.

---

# Phase 4 – Learn Container App Concepts

---

# Lab 4 – Understand Revisions

Deploy new image.

```bash
az containerapp update \
-n app-nginx \
-g rg-containerapp-dev \
--image nginx:latest
```

View revisions:

```bash
az containerapp revision list \
-n app-nginx \
-g rg-containerapp-dev
```

Learn:

* Single revision mode
* Multiple revision mode
* Traffic splitting

Portal:

```text
Container App
      ↓
Revisions
```

---

# Lab 5 – Traffic Splitting

Example:

```text
Revision1 → 80%
Revision2 → 20%
```

CLI:

```bash
az containerapp ingress traffic set \
-n app-nginx \
-g rg-containerapp-dev \
--revision-weight rev1=80 rev2=20
```

---

# Phase 5 – Scaling (Very Important)

Spend at least 2 days here.

---

# Lab 6 – HTTP Autoscaling

Portal:

```text
Scale
```

Set:

```text
Min Replicas = 0
Max Replicas = 10
Concurrent Requests = 50
```

CLI:

```bash
az containerapp update \
-n app-nginx \
-g rg-containerapp-dev \
--min-replicas 0 \
--max-replicas 10
```

---

# Lab 7 – Scale to Zero

Observe:

```text
Replicas = 0
```

Generate traffic:

```bash
for i in {1..500}
do
curl <app-url>
done
```

Watch scaling.

---

# Lab 8 – Queue Based Scaling

Create:

* Storage Queue
* Service Bus Queue

Architecture:

```text
Queue
   ↓
Container App
```

Create scaling rule:

```bash
az containerapp update \
--scale-rule-name queue-rule
```

Learn KEDA concepts.

---

# Phase 6 – Secrets and Environment Variables

---

# Lab 9 – Environment Variables

```bash
az containerapp update \
--set-env-vars APP_ENV=DEV
```

---

# Lab 10 – Secrets

```bash
az containerapp secret set \
-n app-nginx \
-g rg-containerapp-dev \
--secrets dbpassword=Password123
```

Use secret:

```bash
az containerapp update \
--secret-volume-mount
```

Portal:

```text
Container App
      ↓
Secrets
```

---

# Phase 7 – Managed Identity

This is very important in Azure.

Learn:

```text
Container App
      ↓
Managed Identity
      ↓
Key Vault
      ↓
Storage
```

Create identity:

```bash
az containerapp identity assign \
-n app-nginx \
-g rg-containerapp-dev \
--system-assigned
```

Grant access to:

* Key Vault
* Storage Account
* Service Bus

---

# Phase 8 – Networking

Spend at least 2 days.

Topics:

### External Ingress

```text
Internet → Container App
```

### Internal Ingress

```text
VNET → Container App
```

### VNET Integration

### Private Endpoints

### Custom Domains

### Certificates

Portal:

```text
Networking
```

CLI:

```bash
az containerapp ingress enable
```

---

# Phase 9 – Logging and Monitoring

---

## Log Analytics

Portal:

```text
Monitoring
```

CLI:

```bash
az monitor log-analytics query
```

Learn KQL.

Example:

```kql
ContainerAppConsoleLogs_CL
| order by TimeGenerated desc
```

Important tables:

* ContainerAppConsoleLogs_CL
* ContainerAppSystemLogs_CL

---

# Phase 10 – Container App Jobs

Very important.

Equivalent to:

```text
Kubernetes CronJobs
```

Examples:

* Daily Backup
* Report Generation
* Batch Jobs

Create:

```bash
az containerapp job create
```

Learn:

1. Manual Jobs
2. Scheduled Jobs
3. Event Driven Jobs

---

# Phase 11 – Dapr Integration

Learn:

```text
Service Discovery
Pub/Sub
State Store
Secrets
Tracing
```

Architecture:

```text
App1
 ↓
Dapr
 ↓
App2
```

---

# Phase 12 – CI/CD

Since you already know GitLab and Azure DevOps, practice:

## GitLab Pipeline

```text
Build Docker Image
        ↓
Push to ACR
        ↓
Deploy to Container App
```

CLI:

```bash
az containerapp update \
--image myacr.azurecr.io/app:v1
```

---

# Phase 13 – Production Architecture

Build this complete project.

![Image](https://images.openai.com/static-rsc-4/f7_cJV24_8XUL2PJ9-JrEWr1SQcfVNtqY4zQC-vypS4TvcJhJPu6P94kQYlpngbSx2j8L9jxGCqXtYChGSdbHOXzI6PPgsG96n34GFX4ZLtv2-C8FvYoBA-PKjQczoHLWWwD7wC7FiayHRSHRN1s-dk3aT5zyYVhB9lb8YHpabousxv3kE6lZw_iGwVg61mq?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/0pxKgMv4BfTsxx-Y7GJbVL8rltb0gsiE6lvArs8xdQeukv5GbTn_NN5r3wBaxV0Op2d4k6THjrecQJiUpH9aVhbABucoZsYX2_mHVXVJjDl_Z8nFjoUpBBGMzrthEEzeECS7zplAlqHQGlih3Gtfn8FnnwCNEtwjBTfsB2Mr5L52TSvy3Vu6qp6D2AiWPhnV?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ByNnlIf1U2HtEmW0zw_59F_H7Yq4HI2tt3-OpkjAIQ3V5Bri2YX3C495xdB5IpvQmTUpX9FwxHwOBhZH8CjJgQtsoTUmawfhiVimWerxDsK9yY6r11pYRDPLGUOrIrA9jFYPXgYTrcVQ7Ln0A9GmrRn-tmdqUVpK-RHThB1rrPfzE1Z6u89uqFblsCNoRH0G?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/yFnjeQujnafskSEm_dVSGaoHqcHqOoNptkWs3cpASuKUq-PLxbeFZhSbIkWQojSgPSSDkiSxcKy5WLPz76gCKD4CrJ26JnwSmTR3XV-vWKuZ05fPEkYpe4gsCmWzO_9-nph1reEA1VbIJL3Q5bpebZd_wIo0SBdCl0VAwCZcFG-MT7weiMk43zIpcWUvXkwT?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/niCF9EyU5dhqOx01q0RB3ShCwm4rWAcePhlpFwCeVL8nb74zzCnAdnVAAKirB90xilLm0qMHnGYM6XPaHI3cHcuH7LhxET6HVWhStFSoBsKzk23j8O17Pwxx-EIxPuztYU4Q_Jx98-xcfOX1gs5ed2X1rYNPUYK_cSGD5zplzizOD5398gxT_zoxtVod10Ls?purpose=fullsize)

```text
Internet
     ↓
Front Door
     ↓
Container Apps
     ↓
Service Bus
     ↓
Storage
     ↓
Key Vault
```

---

# Suggested Hands-On Projects

## Project 1

Deploy:

```text
Nginx Application
```

Learn:

* Environment
* Scaling
* Logs

---

## Project 2

Deploy:

```text
.NET Web API
```

Learn:

* Revisions
* Traffic Splitting
* Managed Identity

---

## Project 3

Deploy:

```text
Microservices Banking Application
```

Services:

```text
customer-service
account-service
payment-service
notification-service
```

Learn:

* Dapr
* Internal communication
* Service discovery

---

## Project 4

Create:

```text
GitLab CI/CD → ACR → Container Apps
```

---

# CLI Commands You Must Practice

```bash
az containerapp create
az containerapp update
az containerapp delete
az containerapp show
az containerapp list
az containerapp logs show
az containerapp revision list
az containerapp replica list
az containerapp ingress enable
az containerapp secret set
az containerapp identity assign
az containerapp job create
az containerapp env create
```

---

# 15-Day Learning Plan

| Day | Topic                     |
| --- | ------------------------- |
| 1   | Architecture and Concepts |
| 2   | Portal Deployment         |
| 3   | Azure CLI Basics          |
| 4   | Container App Environment |
| 5   | Deploy Applications       |
| 6   | Revisions                 |
| 7   | Traffic Splitting         |
| 8   | Autoscaling               |
| 9   | KEDA Scaling              |
| 10  | Secrets and Identity      |
| 11  | Networking                |
| 12  | Monitoring and Logs       |
| 13  | Container App Jobs        |
| 14  | CI/CD                     |
| 15  | Production Project        |

---

# Recommended Learning Sequence

```text
Docker
   ↓
Azure Container Registry (ACR)
   ↓
Azure Container Apps
   ↓
Container Apps Jobs
   ↓
Dapr
   ↓
CI/CD
   ↓
AKS
```

This sequence works very well because Container Apps teaches many Kubernetes concepts (containers, scaling, ingress, revisions) without exposing all Kubernetes complexity. Once comfortable, moving to AKS becomes much easier.
