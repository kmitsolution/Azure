# AKS Operational Runbooks, Backup & Recovery, and Production Support Processes

These are the next important topics after **AKS Monitoring, Alerting, Cluster Health, Capacity Planning, and Resource Optimization**.

They focus on what happens **when something goes wrong in production**.

```text
AKS Production Operations
        │
        ├── Monitoring
        │
        ├── Alerting
        │
        ├── Operational Runbooks
        │
        ├── Backup & Recovery
        │
        └── Production Support
```

---

# 1. Operational Runbooks

## What is a Runbook?

A **runbook** is a documented, step-by-step procedure used by an operations or support team to handle a known problem.

Instead of an engineer thinking:

> "What should I do now?"

the runbook provides:

```text
Problem
   ↓
Identify
   ↓
Check
   ↓
Fix
   ↓
Verify
   ↓
Document
```

### Example

Alert:

```text
Pod CrashLoopBackOff
```

Runbook:

```text
1. Identify the pod
2. Check pod status
3. Check pod logs
4. Describe the pod
5. Identify the root cause
6. Fix the problem
7. Verify the pod
8. Close the incident
```

---

# 2. Why Runbooks Are Important

Without runbooks:

```text
Alert
 ↓
Engineer investigates from scratch
 ↓
Different engineers follow different procedures
 ↓
Longer resolution time
```

With runbooks:

```text
Alert
 ↓
Known procedure
 ↓
Standard troubleshooting
 ↓
Faster recovery
```

Runbooks are especially useful for:

* Production incidents
* On-call support
* Repeated problems
* New support engineers
* Disaster recovery
* Operational procedures

---

# 3. AKS Runbook Structure

A good runbook should contain:

```text
Runbook Name
Purpose
Symptoms
Impact
Prerequisites
Troubleshooting Steps
Resolution
Verification
Rollback
Escalation
Post-Incident Actions
```

For example:

```text
Runbook:
AKS Pod CrashLoopBackOff
```

---

# 4. Runbook Example — CrashLoopBackOff

## Step 1 — Identify the pod

```bash
kubectl get pods -A
```

Example:

```text
NAME              READY   STATUS
payment-api       0/1     CrashLoopBackOff
```

---

## Step 2 — Check pod details

```bash
kubectl describe pod payment-api
```

Look at:

```text
Events
State
Last State
Restart Count
```

---

## Step 3 — Check logs

```bash
kubectl logs payment-api
```

If the pod has multiple containers:

```bash
kubectl logs payment-api -c <container-name>
```

For the previous crashed container:

```bash
kubectl logs payment-api --previous
```

This is extremely useful for `CrashLoopBackOff`.

---

## Step 4 — Identify the root cause

Possible causes:

```text
Application error
Missing environment variable
Incorrect configuration
Database unavailable
Secret missing
Wrong image
Port configuration
Permission problem
```

---

## Step 5 — Fix

For example, if an environment variable is missing:

```text
Deployment
    ↓
Environment variable
    ↓
Correct value
    ↓
New deployment
```

---

## Step 6 — Verify

```bash
kubectl get pods
```

Expected:

```text
payment-api   1/1   Running
```

Check logs:

```bash
kubectl logs payment-api
```

---

# 5. Runbook Example — ImagePullBackOff

Problem:

```text
Pod
 ↓
ImagePullBackOff
```

### Step 1

```bash
kubectl get pods
```

### Step 2

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
Failed to pull image
```

Possible causes:

```text
Wrong image name
Wrong image tag
Private registry authentication
Image doesn't exist
Network problem
```

### Step 3

Correct the image:

```bash
kubectl edit deployment <deployment-name>
```

Or update the deployment through your normal deployment pipeline.

### Step 4

Verify:

```bash
kubectl get pods
```

---

# 6. Runbook Example — Node NotReady

Check:

```bash
kubectl get nodes
```

Example:

```text
NAME       STATUS
node-001   Ready
node-002   NotReady
node-003   Ready
```

Then:

```bash
kubectl describe node node-002
```

Check:

```text
Conditions
Events
MemoryPressure
DiskPressure
Network problems
```

Also check workloads:

```bash
kubectl get pods -A -o wide
```

The objective is to determine:

```text
Node problem
     ↓
Which workloads are affected?
     ↓
Can they run on another node?
     ↓
Is the node recoverable?
```

---

# 7. Runbook Example — Service Not Accessible

Suppose:

```text
Application URL
      ↓
Not responding
```

Check in this order:

```text
Ingress
   ↓
Service
   ↓
Endpoints
   ↓
Pods
   ↓
Application
```

Commands:

```bash
kubectl get ingress -A
kubectl get svc -A
kubectl get endpoints -A
kubectl get pods -A
```

Then check:

```bash
kubectl describe svc <service-name>
```

and:

```bash
kubectl logs <pod-name>
```

This gives you a systematic troubleshooting process.

---

# 8. Backup and Recovery

Monitoring tells us that something is wrong.

Backup and recovery answer:

> **How do we recover our data and application state?**

There are several different things to consider in AKS.

```text
AKS Recovery
│
├── Kubernetes configuration
├── Application configuration
├── Secrets
├── Persistent data
└── External databases
```

---

# 9. Important: AKS Is Not Your Application Backup

A common misconception is:

> "If I have an AKS cluster, my application is automatically backed up."

Not necessarily.

Your application may depend on:

```text
AKS
+
Azure SQL
+
Azure Database for PostgreSQL
+
Azure Storage
+
Persistent Volumes
+
Key Vault
```

You need a backup strategy for the **entire application architecture**.

---

# 10. What Should We Back Up?

Consider:

### Kubernetes resources

```text
Deployments
Services
Ingress
ConfigMaps
Secrets
Namespaces
RBAC
PersistentVolumeClaims
```

### Application data

For example:

```text
PostgreSQL
Azure SQL
Redis
Storage
```

### Persistent volumes

If applications store data on persistent volumes, that data needs an appropriate backup strategy.

---

# 11. Kubernetes Configuration Backup

Because Kubernetes manifests are normally stored in Git, the preferred approach is often:

```text
Git Repository
      │
      ├── deployment.yaml
      ├── service.yaml
      ├── ingress.yaml
      ├── configmap.yaml
      └── other manifests
```

Then:

```text
Git
 ↓
CI/CD
 ↓
AKS
```

This is an important **GitOps / Infrastructure-as-Code** principle.

---

# 12. Example Kubernetes Backup

You can export resources for troubleshooting or recovery.

For example:

```bash
kubectl get deployment nginx -o yaml
```

Export:

```bash
kubectl get deployment nginx -o yaml > nginx-deployment.yaml
```

Service:

```bash
kubectl get service nginx -o yaml > nginx-service.yaml
```

Ingress:

```bash
kubectl get ingress nginx -o yaml > nginx-ingress.yaml
```

However, don't treat raw `kubectl get -o yaml` output as a perfect disaster-recovery backup. It can contain cluster-generated fields that aren't appropriate for redeployment.

For production, maintain clean manifests in source control.

---

# 13. Azure Backup for AKS

For production environments, Azure provides **Azure Backup for AKS** capabilities for protecting supported AKS resources and persistent volumes.

The general architecture is:

```text
             AKS
              │
      ┌───────┴────────┐
      │                │
 Kubernetes         Persistent
 Resources           Volumes
      │                │
      └───────┬────────┘
              ▼
        Azure Backup
              │
              ▼
       Recovery Point
```

The exact supported resources and configuration depend on the AKS and storage setup, so production implementation should be validated against the current Azure Backup documentation.

---

# 14. Backup vs Recovery

These two terms are different.

### Backup

Creating a copy of data/configuration:

```text
Production
    ↓
Backup
```

### Recovery

Using the backup to restore the environment:

```text
Backup
   ↓
Restore
   ↓
Recovered environment
```

---

# 15. RPO and RTO

Two important production concepts are:

## RPO — Recovery Point Objective

RPO answers:

> **How much data can we afford to lose?**

Example:

```text
RPO = 1 hour
```

If a failure occurs, the organization accepts up to approximately one hour of data loss, depending on the backup design.

---

## RTO — Recovery Time Objective

RTO answers:

> **How quickly must the application be restored?**

Example:

```text
RTO = 2 hours
```

The recovery process should target restoration within that timeframe.

---

# 16. RPO vs RTO

| Concept | Question                     |
| ------- | ---------------------------- |
| RPO     | How much data can we lose?   |
| RTO     | How quickly must we recover? |

Example:

```text
Database failure
      │
      ├── RPO → Maximum acceptable data loss
      │
      └── RTO → Maximum acceptable recovery time
```

---

# 17. Disaster Recovery

A production AKS environment should also consider a regional failure.

Example:

```text
Primary Region
Central India
      │
      │ Disaster
      ▼
Secondary Region
Another Azure Region
```

Depending on the application architecture, recovery may involve:

```text
AKS
Databases
Storage
DNS
Container Registry
Secrets
Networking
```

A disaster recovery design should be tested rather than assumed to work.

---

# 18. Production Support Processes

Now let's discuss how a production team handles incidents.

A typical process is:

```text
Monitoring
   ↓
Alert
   ↓
Incident
   ↓
Investigation
   ↓
Mitigation
   ↓
Resolution
   ↓
Verification
   ↓
Post-Incident Review
```

---

# 19. Incident Management

Suppose Azure Monitor sends:

```text
ALERT:
Production API unavailable
```

The support engineer should not immediately start changing resources randomly.

Instead:

```text
1. Acknowledge
2. Assess impact
3. Investigate
4. Mitigate
5. Resolve
6. Verify
7. Document
```

---

# 20. Step 1 — Acknowledge

First determine:

```text
What happened?
When did it start?
Which application?
Which environment?
How many users affected?
```

Example:

```text
Production
Payment API
Started: 10:15 AM
Impact: Payment requests failing
```

---

# 21. Step 2 — Assess Impact

Determine the severity.

Example:

```text
Production
    │
    ├── One pod affected?
    │
    ├── One service affected?
    │
    └── Entire application unavailable?
```

A single failed pod is very different from a complete production outage.

---

# 22. Step 3 — Investigate

Start with a structured approach.

### Cluster

```bash
kubectl get nodes
```

### Pods

```bash
kubectl get pods -A
```

### Deployments

```bash
kubectl get deployments -A
```

### Services

```bash
kubectl get svc -A
```

### Events

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

### Logs

```bash
kubectl logs <pod-name>
```

---

# 23. Step 4 — Mitigation

Mitigation means reducing the impact while the root cause is being investigated.

For example:

```text
Bad deployment
      ↓
Rollback
```

Check rollout history:

```bash
kubectl rollout history deployment/myapp
```

Rollback:

```bash
kubectl rollout undo deployment/myapp
```

Check:

```bash
kubectl rollout status deployment/myapp
```

Then:

```bash
kubectl get pods
```

---

# 24. Step 5 — Resolution

Once the root cause is identified, implement the permanent fix.

For example:

```text
Root cause:
Incorrect environment variable

Fix:
Correct application configuration

Deployment
    ↓
New version
    ↓
Verification
```

Production changes should normally go through the organization's approved change/deployment process.

---

# 25. Step 6 — Verification

After the fix:

```bash
kubectl get pods
```

Check application health.

For example:

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

Then verify:

```text
Application URL
API health endpoint
Application logs
Azure Monitor
Alerts
```

The important point is:

> **Don't close an incident just because the pod is Running. Verify that the application actually works.**

---

# 26. Step 7 — Post-Incident Review

After a significant production incident, document:

```text
What happened?
When did it happen?
What was the impact?
What was the root cause?
How was it fixed?
How long did recovery take?
How can we prevent it?
```

This is commonly called a:

**Post-Incident Review** or **Postmortem**.

---

# 27. Production Support Escalation

A typical support structure might be:

```text
Alert
  │
  ▼
L1 Support
  │
  ├── Known issue?
  │       │
  │       └── Runbook
  │
  ▼
L2 / DevOps
  │
  ▼
L3 / Application Team
  │
  ▼
Engineering / Vendor
```

The exact structure varies by organization.

---

# 28. Example Production Incident

Let's put everything together.

### Problem

```text
Azure Monitor Alert

Production API unavailable
```

### Investigation

```bash
kubectl get pods -n production
```

Result:

```text
api-7d8f9   0/1   CrashLoopBackOff
```

Check logs:

```bash
kubectl logs api-7d8f9
```

Result:

```text
Database connection failed
```

Check recent deployment:

```bash
kubectl rollout history deployment/api -n production
```

Suppose the problem started immediately after the latest deployment.

### Mitigation

Rollback:

```bash
kubectl rollout undo deployment/api -n production
```

Check:

```bash
kubectl rollout status deployment/api -n production
```

Then:

```bash
kubectl get pods -n production
```

Application becomes healthy.

### Post-Incident

Document:

```text
Root Cause:
Incorrect database configuration in new release

Impact:
Production API unavailable

Mitigation:
Rolled back deployment

Permanent Fix:
Corrected configuration and added validation

Prevention:
Add configuration validation to CI/CD
```

---

# 29. Operational Runbook Example

A production runbook could look like this:

## RUNBOOK: AKS API Unavailable

### Symptoms

```text
API returns HTTP 5xx
```

### Check

```bash
kubectl get pods -n production
kubectl get svc -n production
kubectl get ingress -n production
kubectl get events -n production
```

### Logs

```bash
kubectl logs <pod> -n production
```

### If CrashLoopBackOff

```bash
kubectl logs <pod> -n production --previous
kubectl describe pod <pod> -n production
```

### If recent deployment caused issue

```bash
kubectl rollout history deployment/api -n production
```

### Rollback

```bash
kubectl rollout undo deployment/api -n production
```

### Verify

```bash
kubectl rollout status deployment/api -n production
kubectl get pods -n production
```

### Escalate if

```text
Database unavailable
Networking issue
Multiple applications affected
Cluster-wide problem
Unknown root cause
```

---

# 30. Production Support Golden Rules

### Rule 1 — Don't make random changes

Always:

```text
Check
 ↓
Understand
 ↓
Change
 ↓
Verify
```

---

### Rule 2 — Prefer rollback when appropriate

If a known-good deployment exists and a new release is clearly causing the outage:

```bash
kubectl rollout undo deployment/<name>
```

can be an effective mitigation.

---

### Rule 3 — Preserve evidence

Before deleting or modifying resources, collect useful information:

```bash
kubectl describe
kubectl logs
kubectl get events
```

Otherwise, valuable troubleshooting information may disappear.

---

### Rule 4 — Verify after recovery

Don't just check:

```text
Pod = Running
```

Also verify:

```text
Application = Working
```

---

# 31. Complete Production Operations Flow

You can teach the entire topic as:

```text
                  AKS Production
                        │
                        ▼
                  Monitoring
                        │
                        ▼
                    Alert
                        │
                        ▼
                Incident Created
                        │
                        ▼
                Operational Runbook
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
         Investigation        Mitigation
              │                   │
              └─────────┬─────────┘
                        ▼
                    Resolution
                        │
                        ▼
                    Verification
                        │
                        ▼
               Post-Incident Review
                        │
                        ▼
              Preventive Improvements
```

And alongside this:

```text
        Production Data
              │
              ▼
           Backup
              │
              ▼
        Recovery Point
              │
        Disaster/Failure
              │
              ▼
           Restore
              │
              ▼
          Validation
```

---

# 32. Key Interview Points

### What is a runbook?

A documented step-by-step procedure for handling a known operational problem.

### What is RPO?

Maximum acceptable amount of data loss measured in time.

### What is RTO?

Target maximum time to restore service.

### Why are backups important?

To recover data and application state after accidental deletion, corruption, infrastructure failure, or disaster.

### What is a postmortem?

A structured review of an incident to understand the root cause, impact, resolution, and preventive actions.

### What is rollback?

Returning an application to a previously known-good version.

### Why preserve logs during incidents?

They provide evidence needed to identify the root cause.

---

# 33. AKS Operations — Final Picture

You now have a complete operational lifecycle:

```text
                    AKS
                     │
       ┌─────────────┼─────────────┐
       ▼             ▼             ▼
   Monitoring     Alerting      Logging
       │             │             │
       └─────────────┼─────────────┘
                     ▼
              Cluster Health
                     │
                     ▼
             Capacity Planning
                     │
                     ▼
             Resource Optimization
                     │
                     ▼
             Production Support
                     │
            ┌────────┴────────┐
            ▼                 ▼
        Runbooks          Incident Mgmt
            │                 │
            └────────┬────────┘
                     ▼
                Backup/Recovery
                     │
                     ▼
              Disaster Recovery
                     │
                     ▼
              Post-Incident Review
```

This gives you a solid **AKS Production Operations** section for your training course.
