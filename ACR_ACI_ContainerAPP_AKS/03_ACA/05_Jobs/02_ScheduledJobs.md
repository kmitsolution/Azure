# ACA Jobs — Step 2: Scheduled Job

## 1. What is a Scheduled Job?

A scheduled Job runs automatically according to a schedule.

For example:

```text
Every day at 10:00 PM
          ↓
     ACA Job starts
          ↓
     Container runs
          ↓
     Task completes
          ↓
     Container stops
```

Unlike the Manual Job:

```text
User
 ↓
az containerapp job start
 ↓
Job runs
```

A Scheduled Job doesn't require you to manually start it.

---

# 2. Simple Example

Let's create a very simple Alpine Job.

Every **5 minutes**, the Job will:

```text
Start
  ↓
Print message
  ↓
Wait 10 seconds
  ↓
Exit successfully
```

This is a good lab because you can actually observe multiple executions.

---

# 3. Create Scheduled Job

Let's call it:

```text
scheduled-alpine-job
```

### PowerShell

```powershell
az containerapp job create `
  --name scheduled-alpine-job `
  --resource-group MYRG-India `
  --environment con-env `
  --trigger-type Schedule `
  --cron-expression "*/5 * * * *" `
  --image alpine:latest `
  --command sleep `
  --args 10 `
  --cpu 0.25 `
  --memory 0.5Gi `
  --replica-timeout 60 `
  --replica-retry-limit 2
```

### Bash / Linux

```bash
az containerapp job create \
  --name scheduled-alpine-job \
  --resource-group MYRG-India \
  --environment con-env \
  --trigger-type Schedule \
  --cron-expression "*/5 * * * *" \
  --image alpine:latest \
  --command sleep \
  --args 10 \
  --cpu 0.25 \
  --memory 0.5Gi \
  --replica-timeout 60 \
  --replica-retry-limit 2
```

---

# 4. Understand the Cron Expression

We used:

```text
*/5 * * * *
```

This means:

> Run every 5 minutes.

The five fields are:

```text
┌──────── minute
│ ┌────── hour
│ │ ┌──── day of month
│ │ │ ┌── month
│ │ │ │ ┌ day of week
│ │ │ │ │
* * * * *
```

Examples:

| Cron          | Meaning               |
| ------------- | --------------------- |
| `*/5 * * * *` | Every 5 minutes       |
| `0 * * * *`   | Every hour            |
| `0 0 * * *`   | Every day at midnight |
| `0 2 * * *`   | Every day at 2 AM     |
| `0 9 * * 1`   | Every Monday at 9 AM  |

---

# 5. What Happens Automatically?

After creating the Job, you don't need:

```text
az containerapp job start
```

Azure waits for the schedule.

For our example:

```text
10:00
  ↓
Execution 1
  ↓
Alpine
  ↓
sleep 10
  ↓
Succeeded

10:05
  ↓
Execution 2
  ↓
Alpine
  ↓
sleep 10
  ↓
Succeeded

10:10
  ↓
Execution 3
  ↓
Alpine
  ↓
sleep 10
  ↓
Succeeded
```

---

# 6. Check Job Executions

### PowerShell

```powershell
az containerapp job execution list `
  --name scheduled-alpine-job `
  --resource-group MYRG-India `
  --output table
```

### Bash

```bash
az containerapp job execution list \
  --name scheduled-alpine-job \
  --resource-group MYRG-India \
  --output table
```

You should eventually see multiple executions:

```text
Name                         StartTime                  Status
---------------------------  -------------------------  ---------
scheduled-alpine-job-xxxxx   2026-09-17T...             Succeeded
scheduled-alpine-job-yyyyy   2026-09-17T...             Succeeded
```

---

# 7. Check the Job Configuration

### PowerShell

```powershell
az containerapp job show `
  --name scheduled-alpine-job `
  --resource-group MYRG-India `
  --output json
```

### Bash

```bash
az containerapp job show \
  --name scheduled-alpine-job \
  --resource-group MYRG-India \
  --output json
```

You should be able to see the scheduled trigger configuration.

---

# 8. Manual vs Scheduled Job

Now you have two examples:

```text
                 ACA JOBS
                    │
          ┌─────────┴─────────┐
          │                   │
        Manual             Schedule
          │                   │
          ▼                   ▼
     User starts          Cron starts
          │                   │
          ▼                   ▼
      Execution           Execution
          │                   │
          ▼                   ▼
       Container            Container
          │                   │
          ▼                   ▼
       Complete             Complete
```

### Manual

```powershell
az containerapp job start ...
```

You decide **when** to run it.

### Scheduled

```text
Cron
 ↓
Azure automatically starts it
```

Azure decides **when** to run it based on your schedule.

---

# 9. Important Note About Cron

Cron schedules are generally interpreted in **UTC** for Azure Container Apps Jobs.

So if you want:

```text
10:00 PM IST
```

you need to convert that to UTC:

```text
10:00 PM IST
    ↓
4:30 PM UTC
```

Therefore, the cron would be:

```text
30 16 * * *
```

for 10:00 PM IST daily.

For classroom demonstrations, using:

```text
*/5 * * * *
```

is easier because you don't have to wait until a particular time.

---

# 10. Our ACA Jobs Learning Path

You've completed:

### ✅ Step 1 — Manual Job

```text
Create Job
   ↓
Start manually
   ↓
Execution
   ↓
Succeeded
```

Next:

### ✅ Step 2 — Scheduled Job

```text
Cron
   ↓
Automatic execution
   ↓
Container
   ↓
Complete
```

Then I recommend:

```text
Step 3 → Job Environment Variables
             ↓
Step 4 → Job Secrets
             ↓
Step 5 → Parallelism
             ↓
Step 6 → Replica Completion Count
             ↓
Step 7 → KEDA Event-Driven Job
             ↓
Step 8 → Queue-based Job
             ↓
Step 9 → Job YAML
             ↓
Step 10 → Real-world batch-processing example
```

**Next topic: Parallelism and replica completion** is particularly useful because it explains how ACA Jobs can process multiple pieces of work simultaneously.
