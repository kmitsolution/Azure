# AKS Alerting Strategies with Azure Monitor

After **AKS Monitoring**, the next important topic is **Alerting**.

Monitoring tells us:

> "What is happening?"

Alerting tells us:

> "Something needs attention."

A good AKS monitoring solution normally uses **different alerting strategies for different types of problems**.

---

# 1. What is an Alert?

An Azure Monitor alert continuously evaluates a condition.

For example:

```text
AKS Node CPU > 80%
        ↓
Azure Monitor evaluates condition
        ↓
Condition becomes true
        ↓
Alert fires
        ↓
Action Group
        ↓
Email / SMS / Webhook / Automation
```

Example:

```text
CPU > 80% for 5 minutes
```

Instead of continuously checking the portal, Azure Monitor can notify you.

---

# 2. AKS Alerting Architecture

```text
                    AKS Cluster
                        │
             ┌──────────┴──────────┐
             │                     │
           Logs                  Metrics
             │                     │
             ▼                     ▼
       Log Analytics       Azure Monitor Metrics
             │                     │
             │                     │
             ▼                     ▼
       Log Query Alert       Metric Alert
             │                     │
             └──────────┬──────────┘
                        ▼
                  Azure Monitor
                      Alert
                        │
                        ▼
                   Action Group
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
           Email       SMS      Webhook
```

---

# 3. Main Alerting Strategies

For AKS, we can divide alerts into several categories.

| Strategy                | Example                             |
| ----------------------- | ----------------------------------- |
| Resource alerts         | AKS node CPU > 80%                  |
| Availability alerts     | Pod unavailable                     |
| Performance alerts      | High CPU/memory                     |
| Kubernetes event alerts | Pod CrashLoopBackOff                |
| Log-based alerts        | Error message appears               |
| Prometheus alerts       | Kubernetes metric crosses threshold |
| Infrastructure alerts   | Node NotReady                       |
| Cost alerts             | Unexpected resource usage           |

---

# 4. Strategy 1 — CPU Alert

One of the simplest alerts.

Example:

```text
Node CPU > 80%
for 5 minutes
```

Why?

A temporary CPU spike isn't necessarily a problem.

We therefore don't normally alert immediately.

Instead:

```text
CPU > 80%
      │
      ├── 30 seconds → don't alert
      ├── 2 minutes  → don't alert
      └── 5 minutes  → ALERT
```

This reduces unnecessary alerts.

---

# 5. Strategy 2 — Memory Alert

Memory is particularly important in Kubernetes.

Example:

```text
Memory utilization > 85%
for 5 minutes
```

Why?

High memory can eventually result in:

```text
Pod
 ↓
Memory pressure
 ↓
OOM
 ↓
Container restart
```

---

# 6. Strategy 3 — Node NotReady

This is a more important infrastructure alert.

Example:

```text
AKS Node
   ↓
NotReady
   ↓
Alert
```

For example:

```text
Node aks-nodepool1-123456
Status: NotReady
```

This can indicate a problem with the node or its connectivity.

---

# 7. Strategy 4 — Pod Restart Alert

A pod restarting occasionally may not be serious.

But repeated restarts can indicate a real problem.

For example:

```text
Pod
 ↓
Crash
 ↓
Restart
 ↓
Crash
 ↓
Restart
 ↓
Crash
```

Instead of alerting on one restart:

```text
Restart count > 5
within 10 minutes
```

This is a better alerting strategy.

---

# 8. Strategy 5 — CrashLoopBackOff

This is a very useful Kubernetes alert.

Example:

```text
Pod
 ↓
Container starts
 ↓
Application crashes
 ↓
Container restarts
 ↓
Application crashes
 ↓
CrashLoopBackOff
```

An alert can notify the administrator when this condition occurs.

---

# 9. Strategy 6 — Log-Based Alerts

Not every important condition is a metric.

For example, your application might generate:

```text
ERROR Database connection failed
```

or:

```text
ERROR Payment service unavailable
```

We can search logs using KQL.

Example:

```kusto
ContainerLogV2
| where LogMessage contains "ERROR"
| where TimeGenerated > ago(5m)
```

If the query returns records, Azure Monitor can trigger an alert.

---

# 10. Example — Application Error Alert

Suppose our application generates:

```text
ERROR database connection failed
```

We can use:

```kusto
ContainerLogV2
| where LogMessage contains "database connection failed"
| where TimeGenerated > ago(5m)
| summarize ErrorCount = count()
```

Then create a condition such as:

```text
ErrorCount > 5
```

Result:

```text
Application
     ↓
Error logs
     ↓
Log Analytics
     ↓
KQL query
     ↓
ErrorCount > 5
     ↓
Azure Alert
```

---

# 11. Strategy 7 — Prometheus Alerts

If you are using **Azure Monitor managed Prometheus**, you can create alerts based on Kubernetes metrics.

For example:

```text
Pod CPU usage
       ↓
Prometheus
       ↓
Metric
       ↓
Threshold
       ↓
Alert
```

This is especially useful for Kubernetes-specific metrics.

Examples:

```text
High pod CPU

High pod memory

Pod unavailable

High restart count

Node CPU

Node memory

High request rate
```

---

# 12. Strategy 8 — Availability Alerts

Sometimes CPU and memory are normal but the application is unavailable.

For example:

```text
NGINX
  ↓
Service
  ↓
Ingress
  ↓
Application
```

If the application stops responding, an availability alert can detect it.

For externally accessible applications, **Application Insights availability tests** can also be used to test an endpoint from outside the application environment.

Example:

```text
https://myapp.example.com
```

Expected:

```text
HTTP 200
```

If repeated tests fail:

```text
Availability < threshold
        ↓
Alert
```

---

# 13. Strategy 9 — Action Groups

An alert by itself only identifies the problem.

An **Action Group** defines what happens after the alert fires.

Architecture:

```text
Alert Rule
    │
    ▼
Action Group
    │
    ├── Email
    ├── SMS
    ├── Push notification
    ├── Voice
    ├── Webhook
    ├── Azure Function
    ├── Logic App
    └── Automation
```

For training, start with **Email**.

---

# 14. Create an Action Group

You can create an Action Group from:

**Azure Portal → Monitor → Alerts → Action groups**

Click:

**Create**

Example:

```text
Resource Group:
MYRG-India

Action group name:
aks-alert-actions

Display name:
AKS Alerts
```

Then add:

```text
Notification type:
Email/SMS message

Email:
your-email@example.com
```

Save the Action Group.

---

# 15. Create an Alert Rule from the Portal

Go to:

```text
Azure Portal
   ↓
Monitor
   ↓
Alerts
   ↓
Create
   ↓
Alert rule
```

Select your AKS-related resource or the relevant monitoring resource.

---

# 16. Alert Rule Structure

An alert rule has three important parts:

```text
Signal
   ↓
Condition
   ↓
Action
```

For example:

```text
Signal:
CPU Percentage

Condition:
Greater than 80%

Action:
AKS Alert Action Group
```

---

# 17. Example Alert

Let's create a conceptual alert:

```text
Name:
AKS-High-CPU

Condition:
CPU > 80%

Aggregation:
Average

Evaluation:
5 minutes

Action:
AKS Alerts
```

Meaning:

> If average CPU remains above 80% for the configured evaluation period, fire the alert.

---

# 18. Avoid Alert Storms

This is a very important production concept.

Imagine:

```text
100 Pods
   ↓
100 alerts
```

Your team could receive hundreds of notifications.

Instead, design alerts around **meaningful conditions**.

Bad:

```text
Every pod restart → alert
```

Better:

```text
Pod restarts > 5
within 10 minutes
```

Even better in some environments:

```text
Pod restart rate exceeds expected baseline
```

The exact threshold depends on the workload.

---

# 19. Alert Severity

Azure Monitor alerts support severity levels.

A common way to organize them is:

```text
Sev 0 → Critical
Sev 1 → Error
Sev 2 → Warning
Sev 3 → Informational
Sev 4 → Verbose
```

For example:

### Critical

```text
Production application unavailable
```

### Error

```text
Node NotReady
```

### Warning

```text
CPU > 80%
```

### Informational

```text
Deployment completed
```

The exact severity assignment should be based on your operational requirements.

---

# 20. Threshold-Based vs Dynamic Alerts

There are two useful approaches.

## Static threshold

Example:

```text
CPU > 80%
```

Simple and easy to understand.

---

## Dynamic threshold

Azure Monitor can use historical behavior to identify unusual values.

Conceptually:

```text
Normal CPU
     │
     │
     │      ┌── unusual spike
     │      │
─────┼──────┼────────
     │      │
     │      │
     └──────┴──────── Time
```

This can be useful when a fixed threshold doesn't represent normal behavior for the workload.

---

# 21. Alert Suppression / Noise Reduction

Suppose a deployment temporarily causes:

```text
CPU = 90%
```

You may not want an alert every time you deploy.

Production environments often use:

```text
Maintenance windows
Alert processing rules
Severity filtering
Aggregation
Evaluation periods
```

to reduce unnecessary notifications.

---

# 22. Alert Processing Rules

An **Alert Processing Rule** can modify how alerts are processed.

For example:

```text
AKS alerts
    ↓
Alert Processing Rule
    ↓
Suppress notifications during maintenance
```

This is useful when planned maintenance is happening.

---

# 23. Recommended AKS Alert Set

For a production AKS environment, you could start with:

### Infrastructure

```text
Node NotReady
Node CPU high
Node memory high
Node disk pressure
```

### Kubernetes

```text
Pod CrashLoopBackOff
Pod restart rate high
Pod unavailable
Deployment replica mismatch
```

### Application

```text
HTTP 5xx high
Application errors high
Availability failure
High latency
```

### Capacity

```text
CPU saturation
Memory saturation
Insufficient capacity
```

### Security

```text
Unexpected authentication failures
Suspicious activity
```

---

# 24. Simple Training Example

For your AKS training, I recommend demonstrating **three alerts**.

### Alert 1 — CPU

```text
CPU > 80%
```

### Alert 2 — Pod Restart

```text
Restart count > 5
```

### Alert 3 — Application Error

```text
ERROR count > 5
```

Then show:

```text
                AKS
                 │
        ┌────────┼────────┐
        │        │        │
       CPU      Pod      Logs
        │      Restart     │
        │        │         │
        └────────┼─────────┘
                 ▼
           Azure Monitor
                 │
                 ▼
            Alert Rule
                 │
                 ▼
            Action Group
                 │
                 ▼
               Email
```

---

# 25. Important Difference: Monitoring vs Alerting

This is a good exam/interview point.

### Monitoring

Answers:

> What is happening?

Example:

```text
CPU = 75%
Memory = 60%
Pod restarts = 2
```

### Alerting

Answers:

> Does somebody need to take action?

Example:

```text
CPU > 80% for 5 minutes
        ↓
ALERT
        ↓
Email administrator
```

---

# 26. Real-World Alerting Strategy

Don't create an alert for every metric.

Instead:

```text
Collect lots of telemetry
          ↓
Analyze telemetry
          ↓
Identify important conditions
          ↓
Create meaningful alerts
          ↓
Route alerts to correct team
          ↓
Take action
```

The goal is **actionable alerts**, not maximum number of alerts.

---

## Next AKS Monitoring Topic

After **Alerting Strategies**, a natural next topic is:

### **Azure Monitor + Log Analytics + KQL for AKS Troubleshooting**

We can build a practical lab where we intentionally create:

1. A `CrashLoopBackOff`
2. An `ImagePullBackOff`
3. A pod with high CPU
4. Application errors
5. Then use **KQL + Azure Monitor** to identify exactly what went wrong.
