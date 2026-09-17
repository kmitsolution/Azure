# Azure Container Apps Jobs — Event-Driven Job with KEDA

In this demo, we will create an **event-driven Azure Container Apps Job** using **KEDA** and an **Azure Storage Queue**.

We will keep the container very simple by using the `alpine:latest` image.

## Architecture

```text
Azure Storage Account
        │
        ▼
   Storage Queue
     workqueue
        │
        │ Messages
        ▼
       KEDA
        │
        ▼
ACA Event-Driven Job
        │
        ▼
   Alpine Container
        │
        ▼
   Process Work
        │
        ▼
    Job Complete
```

---

# 1. What Are We Building?

We will create:

| Resource        | Name               |
| --------------- | ------------------ |
| Resource Group  | `MYRG-India`       |
| ACA Environment | `con-env`          |
| Storage Account | `kmitjobstorage01` |
| Storage Queue   | `workqueue`        |
| ACA Job         | `queue-alpine-job` |
| Container Image | `alpine:latest`    |

The flow will be:

```text
Message arrives in Storage Queue
              ↓
             KEDA
              ↓
     Event-driven Job starts
              ↓
       Alpine container
              ↓
        Process workload
              ↓
        Container exits
              ↓
       Job execution completes
```

---

# 2. What Is KEDA?

**KEDA** stands for:

> Kubernetes Event-driven Autoscaling

KEDA allows applications and jobs to react to external events.

Examples include:

* Azure Storage Queue
* Azure Service Bus
* Kafka
* RabbitMQ
* Redis
* Other event sources

For our example:

```text
Storage Queue
     ↓
Messages
     ↓
KEDA
     ↓
ACA Job
```

---

# 3. Why Use an Event-Driven Job?

Consider an application that needs to process orders.

```text
Order 101
Order 102
Order 103
Order 104
```

These orders can be placed into a queue.

Instead of keeping a container running continuously:

```text
Container
   ↓
Check queue
   ↓
No work
   ↓
Check again
   ↓
No work
```

we can use an event-driven Job.

```text
Queue has work
      ↓
     KEDA
      ↓
Start Job
      ↓
Process work
      ↓
Finish
```

This is particularly useful for **batch and asynchronous processing**.

---

# 4. Step 1 — Set Variables

## PowerShell

```powershell
$RG="MYRG-India"
$STORAGE="kmitjobstorage01"
$QUEUE="workqueue"
$ENV="con-env"
```

## Bash / Linux

```bash
RG="MYRG-India"
STORAGE="kmitjobstorage01"
QUEUE="workqueue"
ENV="con-env"
```

> **Important:** Storage Account names must be globally unique and contain only lowercase letters and numbers.

If this name is already taken, use another name such as:

```text
kmitjobstorage02
```

---

# 5. Step 2 — Create Storage Account

## PowerShell

```powershell
az storage account create `
  --name $STORAGE `
  --resource-group $RG `
  --location centralindia `
  --sku Standard_LRS
```

## Bash / Linux

```bash
az storage account create \
  --name "$STORAGE" \
  --resource-group "$RG" \
  --location centralindia \
  --sku Standard_LRS
```

---

# 6. Step 3 — Create Storage Queue

Create a queue called:

```text
workqueue
```

## PowerShell

```powershell
az storage queue create `
  --name $QUEUE `
  --account-name $STORAGE `
  --auth-mode login
```

## Bash / Linux

```bash
az storage queue create \
  --name "$QUEUE" \
  --account-name "$STORAGE" \
  --auth-mode login
```

You can verify the queue:

### PowerShell

```powershell
az storage queue list `
  --account-name $STORAGE `
  --auth-mode login `
  --output table
```

### Bash / Linux

```bash
az storage queue list \
  --account-name "$STORAGE" \
  --auth-mode login \
  --output table
```

---

# 7. Step 4 — Get Storage Account Key

When we tried to add messages using:

```text
--auth-mode login
```

Azure returned:

```text
You do not have the required permissions needed to perform this operation.
```

This happens because the signed-in Azure user needs the appropriate Storage Queue RBAC role.

For this simple demonstration, we will use the **Storage Account access key**.

## PowerShell

```powershell
$STORAGE_KEY = az storage account keys list `
  --account-name $STORAGE `
  --resource-group $RG `
  --query "[0].value" `
  --output tsv
```

## Bash / Linux

```bash
STORAGE_KEY=$(az storage account keys list \
  --account-name "$STORAGE" \
  --resource-group "$RG" \
  --query "[0].value" \
  --output tsv)
```

Now:

```text
$STORAGE_KEY
```

contains the Storage Account key.

### Important Note

Do **not**:

* Commit the key to Git
* Put it into source code
* Share it publicly
* Put it directly into a YAML file committed to Git

For production workloads, use stronger identity-based authentication such as **Managed Identity + RBAC** where supported.

---

# 8. Step 5 — Add Messages to the Queue

Now we can add messages using the Storage Account key.

## PowerShell

```powershell
az storage message put `
  --queue-name $QUEUE `
  --content "Task 1" `
  --account-name $STORAGE `
  --account-key $STORAGE_KEY `
  --auth-mode key
```

Add two more:

```powershell
az storage message put `
  --queue-name $QUEUE `
  --content "Task 2" `
  --account-name $STORAGE `
  --account-key $STORAGE_KEY `
  --auth-mode key
```

```powershell
az storage message put `
  --queue-name $QUEUE `
  --content "Task 3" `
  --account-name $STORAGE `
  --account-key $STORAGE_KEY `
  --auth-mode key
```

---

## Bash / Linux

```bash
az storage message put \
  --queue-name "$QUEUE" \
  --content "Task 1" \
  --account-name "$STORAGE" \
  --account-key "$STORAGE_KEY" \
  --auth-mode key
```

```bash
az storage message put \
  --queue-name "$QUEUE" \
  --content "Task 2" \
  --account-name "$STORAGE" \
  --account-key "$STORAGE_KEY" \
  --auth-mode key
```

```bash
az storage message put \
  --queue-name "$QUEUE" \
  --content "Task 3" \
  --account-name "$STORAGE" \
  --account-key "$STORAGE_KEY" \
  --auth-mode key
```

---

# 9. Step 6 — Verify the Queue Messages

We can peek at the messages without removing them from the queue.

## PowerShell

```powershell
az storage message peek `
  --queue-name $QUEUE `
  --account-name $STORAGE `
  --account-key $STORAGE_KEY `
  --num-messages 10
```

## Bash / Linux

```bash
az storage message peek \
  --queue-name "$QUEUE" \
  --account-name "$STORAGE" \
  --account-key "$STORAGE_KEY" \
  --num-messages 10
```

We should see messages similar to:

```text
Task 1
Task 2
Task 3
```

At this point:

```text
Storage Account
      │
      ▼
  workqueue
      │
      ├── Task 1
      ├── Task 2
      └── Task 3
```

---

# 10. Step 7 — Create the Event-Driven ACA Job

Now we need to create the ACA Job with an **event trigger**.

Unlike our Manual Job:

```text
triggerType = Manual
```

or Scheduled Job:

```text
triggerType = Schedule
```

we need:

```text
triggerType = Event
```

The important configuration is:

```text
eventTriggerConfig
```

Conceptually:

```text
Storage Queue
      ↓
KEDA scaler
      ↓
eventTriggerConfig
      ↓
ACA Job
```

---

# 11. Important CLI Version Note

The `containerapp` CLI extension is currently evolving, and **event-driven Container Apps Jobs use preview functionality**.

You can check your installed extension:

### PowerShell / Bash

```bash
az extension show --name containerapp
```

Update it if required:

```bash
az extension update --name containerapp
```

Then check the Job command:

```bash
az containerapp job create --help
```

Look for the event-trigger options.

> **Important:** Don't copy an event-trigger YAML from an older tutorial blindly. The supported `eventTriggerConfig` schema can change with the Container Apps CLI/API version.

---

# 12. Conceptual KEDA Configuration

For our demo, the important information is:

```text
Event source:
Azure Storage Queue

Queue:
workqueue

Trigger:
Queue contains messages

Container:
alpine:latest
```

The architecture becomes:

```text
              Storage Account
                    │
                    ▼
                workqueue
                    │
              Task 1, Task 2
                    │
                    ▼
                  KEDA
                    │
             Queue has work
                    │
                    ▼
             ACA Event Job
                    │
                    ▼
              Alpine Container
                    │
                    ▼
               Process task
                    │
                    ▼
                  Exit
```

---

# 13. What Happens When the Queue Is Empty?

Suppose:

```text
workqueue = 0 messages
```

There is no work to process.

```text
Queue
  │
  └── 0 messages
          ↓
        KEDA
          ↓
     No workload
```

When messages arrive:

```text
Queue
  │
  ├── Task 1
  ├── Task 2
  └── Task 3
          ↓
        KEDA
          ↓
     Work detected
          ↓
      ACA Job
```

---

# 14. Event-Driven vs Scheduled Job

This is an important exam/interview concept.

### Scheduled Job

```text
Cron
 ↓
Job
 ↓
Container
 ↓
Complete
```

Example:

```text
0 22 * * *
```

Run every day at 10 PM.

### Event-Driven Job

```text
Event
 ↓
KEDA
 ↓
Job
 ↓
Container
 ↓
Complete
```

Example:

```text
Queue contains messages
        ↓
       KEDA
        ↓
      Job starts
```

---

# 15. Manual vs Scheduled vs Event

| Trigger  | Starts When                           |
| -------- | ------------------------------------- |
| Manual   | User/application explicitly starts it |
| Schedule | Cron schedule occurs                  |
| Event    | External event/workload is detected   |

Visual:

```text
                    ACA JOB
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
       Manual       Schedule       Event
          │            │            │
          ▼            ▼            ▼
        User          Cron         KEDA
          │            │            │
          └────────────┼────────────┘
                       ▼
                  Job Execution
                       │
                       ▼
                   Container
                       │
                       ▼
                    Complete
```

---

# 16. Real-World Use Cases

Event-driven ACA Jobs are useful for workloads such as:

### Order Processing

```text
Orders
   ↓
Service Bus / Queue
   ↓
KEDA
   ↓
ACA Job
   ↓
Process orders
```

### Image Processing

```text
Images uploaded
      ↓
Queue
      ↓
KEDA
      ↓
ACA Job
      ↓
Resize / compress images
```

### Report Generation

```text
Report request
      ↓
Queue
      ↓
KEDA
      ↓
ACA Job
      ↓
Generate PDF
      ↓
Store PDF
```

### Data Processing

```text
Data files
    ↓
Queue
    ↓
KEDA
    ↓
ACA Job
    ↓
Process data
```

---

# 17. Key Points to Remember

### KEDA

```text
KEDA = Kubernetes Event-driven Autoscaling
```

It allows workloads to respond to external events.

### ACA Event Job

```text
Event
 ↓
KEDA
 ↓
Job execution
 ↓
Container
 ↓
Complete
```

### Storage Queue

Our queue is:

```text
workqueue
```

and currently contains:

```text
Task 1
Task 2
Task 3
```

### Storage Key

Because our initial:

```bash
--auth-mode login
```

operation failed due to missing Storage Queue RBAC permissions, we used:

```text
--auth-mode key
```

with:

```text
STORAGE_KEY
```

for this learning demo.

---

## Next Step

The next hands-on step is to **create the actual `Event`-triggered ACA Job and configure its Azure Storage Queue KEDA scaler**. We'll use your existing `con-env`, `workqueue`, and `STORAGE_KEY`, and verify that adding queue messages causes Job executions to be created.
