# ACA Jobs — Next Topic: Parallelism & Replica Completion Count

Now that you have successfully demonstrated:

1. **Manual Job**
2. **Scheduled Job**
3. **`replicaTimeout`**
4. **`replicaRetryLimit`**

the next important concept is **Parallelism** and **Replica Completion Count**.

---

## 1. What is Parallelism?

**Parallelism** defines how many replicas of a Job can run **at the same time during one Job execution**.

For example:

```text
parallelism = 3
```

means ACA can run up to **3 replicas simultaneously**.

```text
             Job Execution
                  |
       +----------+----------+
       |          |          |
       ↓          ↓          ↓
   Replica 1  Replica 2  Replica 3
       |          |          |
       ↓          ↓          ↓
     Task A     Task B     Task C
```

This is useful when you have many independent pieces of work.

### Real-world example

Imagine you have **100 files** to process.

Instead of:

```text
1 replica
   ↓
100 files
   ↓
Process one by one
```

you could use:

```text
parallelism = 5

Replica 1 → Files 1–20
Replica 2 → Files 21–40
Replica 3 → Files 41–60
Replica 4 → Files 61–80
Replica 5 → Files 81–100
```

The actual work distribution depends on how your application assigns work; ACA doesn't automatically split your business workload into file ranges.

---

# 2. What is Replica Completion Count?

`replicaCompletionCount` specifies **how many successful replicas are required before the Job execution is considered complete**.

For example:

```text
replicaCompletionCount = 3
```

means ACA needs **3 successful replica completions**.

---

# 3. Parallelism vs Completion Count

These two settings are related but different.

| Setting                  | Meaning                                      |
| ------------------------ | -------------------------------------------- |
| `parallelism`            | How many replicas can run simultaneously     |
| `replicaCompletionCount` | How many replicas must successfully complete |

### Example 1

```text
parallelism = 3
replicaCompletionCount = 1
```

ACA can run up to 3 replicas simultaneously, but only **1 successful completion** is required.

### Example 2

```text
parallelism = 3
replicaCompletionCount = 3
```

ACA can run 3 simultaneously and needs **all 3 to successfully complete**.

---

# 4. Simple Demo

Let's modify our Alpine Job.

Currently you have:

```text
parallelism = 1
replicaCompletionCount = 1
```

Let's create a new Job:

```text
parallel-alpine-job
```

with:

```text
parallelism = 3
replicaCompletionCount = 3
```

Each Alpine replica will simply sleep for 5 seconds and then exit successfully.

---

## 5. Create the Parallel Job

### PowerShell

```powershell
az containerapp job create `
  --name parallel-alpine-job `
  --resource-group MYRG-India `
  --environment con-env `
  --trigger-type Manual `
  --image alpine:latest `
  --command sleep `
  --args 5 `
  --cpu 0.25 `
  --memory 0.5Gi `
  --parallelism 3 `
  --replica-completion-count 3 `
  --replica-timeout 60 `
  --replica-retry-limit 2
```

### Bash / Linux

```bash
az containerapp job create \
  --name parallel-alpine-job \
  --resource-group MYRG-India \
  --environment con-env \
  --trigger-type Manual \
  --image alpine:latest \
  --command sleep \
  --args 5 \
  --cpu 0.25 \
  --memory 0.5Gi \
  --parallelism 3 \
  --replica-completion-count 3 \
  --replica-timeout 60 \
  --replica-retry-limit 2
```

---

# 6. Start the Job

### PowerShell

```powershell
az containerapp job start `
  --name parallel-alpine-job `
  --resource-group MYRG-India
```

### Bash / Linux

```bash
az containerapp job start \
  --name parallel-alpine-job \
  --resource-group MYRG-India
```

---

# 7. What Should Happen?

The execution starts:

```text
             Execution
                 |
       parallelism = 3
                 |
     +-----------+-----------+
     |           |           |
     ↓           ↓           ↓
 Replica 1   Replica 2   Replica 3
   sleep 5     sleep 5     sleep 5
     ↓           ↓           ↓
 Succeeded   Succeeded   Succeeded
     \           |           /
      \          |          /
       +---------+---------+
                 ↓
     completion count = 3
                 ↓
        Execution Succeeded
```

---

# 8. Check the Execution

### PowerShell

```powershell
az containerapp job execution list `
  --name parallel-alpine-job `
  --resource-group MYRG-India `
  --output table
```

### Bash

```bash
az containerapp job execution list \
  --name parallel-alpine-job \
  --resource-group MYRG-India \
  --output table
```

You should see:

```text
Name                    Status
----------------------  ---------
parallel-alpine-job-... Succeeded
```

---

# 9. Check the Execution Details

### PowerShell

```powershell
az containerapp job execution show `
  --name parallel-alpine-job `
  --resource-group MYRG-India `
  --job-execution-name <EXECUTION_NAME> `
  --output json
```

### Bash

```bash
az containerapp job execution show \
  --name parallel-alpine-job \
  --resource-group MYRG-India \
  --job-execution-name <EXECUTION_NAME> \
  --output json
```

This is where you can demonstrate that the execution involved multiple replicas.

---

# 10. Very Important Concept

Don't confuse these:

```text
parallelism
```

with:

```text
replicaCompletionCount
```

Think of it this way:

**Parallelism = How many workers can work simultaneously?**

**Completion count = How many workers need to successfully finish?**

For example:

```text
parallelism = 5
completion count = 5
```

means:

```text
5 workers
   ↓
work simultaneously
   ↓
5 successful completions
   ↓
Job execution succeeds
```

Whereas:

```text
parallelism = 5
completion count = 2
```

means ACA can run up to 5 replicas concurrently, but only 2 successful completions are required for the execution to complete.

---

## ACA Jobs Learning Path

You've now covered:

```text
✅ Manual Jobs
       ↓
✅ Scheduled Jobs
       ↓
✅ replicaTimeout
       ↓
✅ replicaRetryLimit
       ↓
➡ Parallelism
       ↓
➡ Replica Completion Count
       ↓
🔜 Event-driven Jobs
       ↓
🔜 KEDA
       ↓
🔜 Environment Variables
       ↓
🔜 Secrets
       ↓
🔜 Job YAML
       ↓
🔜 Real-world batch processing
```

**The next major topic after this should be Event-driven Jobs with KEDA**, where the Job starts based on an external event such as messages arriving in a queue.
