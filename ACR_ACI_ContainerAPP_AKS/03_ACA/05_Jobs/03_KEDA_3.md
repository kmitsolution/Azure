# Azure Container Apps Jobs – Event-Driven Jobs with KEDA

## 1. Introduction

Azure Container Apps Jobs can run containers when a particular event occurs.

For example:

```text
Azure Storage Queue
        ↓
      KEDA
        ↓
Container Apps Job
        ↓
Container
        ↓
Process the task
        ↓
Job completes
```

This is called an **event-driven Container Apps Job**.

A common real-world use case is:

> Whenever messages are available in an Azure Storage Queue, automatically start Container Apps Job executions to process those messages.

---

# 2. What is KEDA?

**KEDA** stands for:

**Kubernetes Event-driven Autoscaling**

Azure Container Apps uses KEDA internally to monitor event sources and determine when Job executions should be started.

For our example:

```text
Azure Storage Queue
        │
        │ Number of messages
        ▼
       KEDA
        │
        │ Start executions
        ▼
Container Apps Job
```

KEDA is responsible for **detecting the workload**.

It is **not responsible for processing or deleting the queue messages**.

---

# 3. Our Demo Architecture

We are going to use:

```text
Azure Storage Account
        │
        └── Queue: workqueue
                 │
                 │ Task 1
                 │ Task 2
                 │ Task 3
                 ▼
              KEDA
                 │
                 ▼
       Container Apps Job
         queue-alpine-job
                 │
                 ▼
            Alpine Container
```

Our resources:

| Resource                   | Value              |
| -------------------------- | ------------------ |
| Resource Group             | `MYRG-India`       |
| Container Apps Environment | `con-env`          |
| Storage Account            | `kmitjobstorage01` |
| Storage Queue              | `workqueue`        |
| Job                        | `queue-alpine-job` |
| Container Image            | `alpine:latest`    |
| Scale Rule                 | `queueJob`         |
| Scale Rule Type            | `azure-queue`      |

---

# 4. Create the Storage Account

We created a Storage Account for the demo.

Example:

```text
Storage Account
    kmitjobstorage01
```

Inside the Storage Account we created a queue:

```text
Queues
   │
   └── workqueue
```

The queue will contain messages such as:

```text
Task 1
Task 2
Task 3
```

---

# 5. Add Messages to the Queue

Using Azure Portal, open:

**Storage Account → Data storage → Queues → workqueue**

Then add messages.

For example:

```text
Task 1
Task 2
Task 3
```

The queue now looks like:

```text
workqueue
──────────────
Task 1
Task 2
Task 3
```

---

# 6. Create an Event-Driven Container Apps Job

Open:

**Azure Portal → Container Apps Jobs → Create**

### Basic configuration

Use:

```text
Job name:
queue-alpine-job
```

Select the existing environment:

```text
con-env
```

For trigger type select:

```text
Event
```

This is important.

We are not creating:

```text
Manual
```

or:

```text
Schedule
```

We are creating:

```text
Event
```

because KEDA will monitor an external event source.

---

# 7. Configure Replica Settings

For our simple demonstration:

### Replica timeout

```text
60 seconds
```

### Replica retry limit

```text
2
```

### Replica completion count

```text
1
```

### Parallelism

```text
1
```

### Minimum executions

```text
0
```

### Maximum executions

```text
3
```

### Polling interval

```text
30 seconds
```

The important settings are:

```text
Min executions = 0
Max executions = 3
Polling interval = 30 seconds
```

---

# 8. What Does Minimum Execution = 0 Mean?

When there are no queue messages:

```text
Queue = 0
       ↓
KEDA
       ↓
0 Job executions
```

This means the Job does not continuously run when there is no work.

This is one of the important benefits of event-driven Jobs.

---

# 9. What Does Maximum Execution = 3 Mean?

Suppose the queue has many messages.

We configured:

```text
Maximum executions = 3
```

This limits how many Job executions can be started by the scaler according to the configured scaling behavior.

Conceptually:

```text
Queue
  │
  ├── Task 1
  ├── Task 2
  ├── Task 3
  ├── Task 4
  └── Task 5
       │
       ▼
      KEDA
       │
       ├── Execution 1
       ├── Execution 2
       └── Execution 3
```

---

# 10. Configure the Container

For our simple demonstration, we use:

```text
Image source:
Docker Hub or other registries
```

Select:

```text
Public
```

Registry:

```text
docker.io
```

Image:

```text
alpine:latest
```

We deliberately selected Alpine because it is a very small image and easy to understand.

---

# 11. First Alpine Test

Initially, we used:

```text
Image:
alpine:latest
```

with:

```text
Command:
sleep

Arguments:
5
```

The intention was:

```text
Alpine starts
     ↓
sleep 5
     ↓
Exit
```

However, for a clean demonstration, a simpler command is:

```text
Command:
echo
```

Arguments:

```text
Hello from Queue Job
```

The container performs:

```text
echo "Hello from Queue Job"
       ↓
Exit code 0
       ↓
Succeeded
```

This makes it easy to verify that the Job itself works.

---

# 12. Configure the KEDA Scale Rule

Go to:

**Scale rules → Add**

The portal may show predefined scalers such as:

```text
Azure Service Bus
Azure Pipelines
...
```

For Azure Storage Queue, use:

### Blank form

Click:

**Blank form → Apply**

---

# 13. Scale Rule Name

Enter:

```text
queueJob
```

This is simply a meaningful name for our scale rule.

---

# 14. Custom Rule Type

Enter:

```text
azure-queue
```

This is the KEDA scaler type for Azure Storage Queue.

So:

```text
Rule name:
queueJob

Custom rule type:
azure-queue
```

---

# 15. Configure Metadata

Add the following metadata.

### Metadata 1

```text
Name:
accountName

Value:
kmitjobstorage01
```

### Metadata 2

```text
Name:
queueName

Value:
workqueue
```

### Metadata 3

```text
Name:
queueLength

Value:
1
```

Final configuration:

```text
accountName = kmitjobstorage01
queueName   = workqueue
queueLength = 1
```

---

# 16. What Does `queueLength = 1` Mean?

This is the target queue length used by the scaler.

For our teaching example:

```text
Queue messages = 0
        ↓
No work

Queue messages >= 1
        ↓
KEDA detects work
        ↓
Job execution can be started
```

It does **not** mean that KEDA deletes one message.

It is a scaling parameter.

---

# 17. Configure Authentication

The scale rule needs access to the Storage Queue.

In the Authentication section we saw:

```text
Secret reference
Trigger parameter
```

We selected a secret:

```text
sec1
```

For example:

```text
Secret reference:
sec1
```

Then the **Trigger parameter** must be:

```text
connection
```

Therefore:

```text
Secret reference      Trigger parameter
---------------------------------------
sec1                  connection
```

---

# 18. What is the Trigger Parameter?

The trigger parameter tells the KEDA scaler which authentication parameter to use.

For the Azure Queue scaler:

```text
connection
```

is used for the Storage connection information.

So do **not** enter:

```text
sec1
```

in the Trigger parameter field.

Use:

```text
connection
```

The relationship is:

```text
Secret
 sec1
  │
  │ referenced as
  ▼
connection
  │
  ▼
KEDA azure-queue scaler
```

---

# 19. Secret vs Environment Variable

This was an important point we discovered while using the Portal.

When editing the container, you may see:

```text
Properties
Environment variables
Health probes
Volume mounts
```

The **Environment variables** tab is for variables used by the container/application.

For example:

```text
APP_ENV=production
```

A Storage connection string used for KEDA authentication should instead be configured as a **secret** and referenced by the scale rule.

Conceptually:

```text
Application environment variable
        ↓
Used by application


KEDA secret
        ↓
Used by KEDA scaler
```

Do not unnecessarily put Storage credentials into ordinary environment variables.

---

# 20. Important: KEDA Does Not Process Queue Messages

This is the most important concept in this demonstration.

Suppose the queue contains:

```text
Task 1
Task 2
Task 3
```

KEDA does:

```text
Check queue
    ↓
Find messages
    ↓
Start Job execution
```

KEDA does **not** do:

```text
Read Task 1
Delete Task 1
Read Task 2
Delete Task 2
```

The application inside the Job must do that.

---

# 21. What Happened When We Clicked "Run Now"?

We clicked:

**Run now**

This manually started a Job execution.

The flow was:

```text
Run Now
   ↓
Start Job execution
   ↓
Alpine container
   ↓
echo "Hello from Queue Job"
   ↓
Exit
   ↓
Succeeded
```

However, our queue still contained:

```text
Task 1
Task 2
Task 3
```

This is **expected**.

Why?

Because our Alpine container never connected to the Storage Queue.

It only executed:

```text
echo "Hello from Queue Job"
```

---

# 22. Why Didn't Task 1 Get Deleted?

Because there is no queue-processing code inside Alpine.

Our container does:

```text
Alpine
  ↓
echo
  ↓
exit
```

It does not do:

```text
Connect to Azure Storage Queue
        ↓
Get message
        ↓
Process message
        ↓
Delete message
```

Therefore:

```text
workqueue
──────────────
Task 1
Task 2
Task 3
```

remains unchanged.

---

# 23. KEDA vs Worker Application

This distinction should be clearly understood.

### KEDA

KEDA answers:

> "Is there work available, and how many Job executions should be started?"

### Worker application

The worker answers:

> "What should I do with the queue message?"

Therefore:

```text
             Azure Storage Queue
                    │
                    │ message count
                    ▼
                  KEDA
                    │
                    │ start execution
                    ▼
          Container Apps Job
                    │
                    ▼
             Worker Container
                    │
                    ├── Read message
                    ├── Process message
                    └── Delete message
```

---

# 24. Why Alpine Was Only a Demo

We deliberately used:

```text
alpine:latest
```

because it is simple and small.

It is useful for demonstrating:

* Event-driven Job creation
* KEDA scale rule
* Queue connection
* Secret reference
* Job execution
* Successful completion

But it is **not a real queue-processing application**.

For a complete production-like demonstration, we need a small worker application.

---

# 25. Real Queue Processing Architecture

The next version should look like this:

```text
┌─────────────────────────┐
│ Azure Storage Account   │
│                         │
│ Queue: workqueue        │
│                         │
│ Task 1                  │
│ Task 2                  │
│ Task 3                  │
└────────────┬────────────┘
             │
             ▼
       ┌───────────┐
       │   KEDA    │
       │           │
       │ azure-    │
       │ queue     │
       └─────┬─────┘
             │
             ▼
┌─────────────────────────┐
│ Container Apps Job      │
│                         │
│ Python Worker           │
└────────────┬────────────┘
             │
             ▼
       Read Task 1
             │
             ▼
      Process Task 1
             │
             ▼
       Delete Task 1
             │
             ▼
       Job completes
```

Then the queue changes from:

```text
Before

Task 1
Task 2
Task 3
```

to:

```text
After

(empty)
```

---

# 26. Azure CLI Equivalent

The same Event Job can also be created using Azure CLI.

## PowerShell

```powershell
az containerapp job create `
  --name queue-alpine-job `
  --resource-group MYRG-India `
  --environment con-env `
  --trigger-type Event `
  --replica-timeout 60 `
  --replica-retry-limit 2 `
  --replica-completion-count 1 `
  --parallelism 1 `
  --polling-interval 30 `
  --min-executions 0 `
  --max-executions 3 `
  --scale-rule-name queueJob `
  --scale-rule-type azure-queue `
  --scale-rule-metadata "accountName=kmitjobstorage01" "queueName=workqueue" "queueLength=1" `
  --scale-rule-auth "connection=sec1" `
  --secrets "sec1=<STORAGE_CONNECTION_STRING>" `
  --image alpine:latest `
  --command echo `
  --args "Hello from Queue Job"
```

### Bash/Linux

```bash
az containerapp job create \
  --name queue-alpine-job \
  --resource-group MYRG-India \
  --environment con-env \
  --trigger-type Event \
  --replica-timeout 60 \
  --replica-retry-limit 2 \
  --replica-completion-count 1 \
  --parallelism 1 \
  --polling-interval 30 \
  --min-executions 0 \
  --max-executions 3 \
  --scale-rule-name queueJob \
  --scale-rule-type azure-queue \
  --scale-rule-metadata "accountName=kmitjobstorage01" "queueName=workqueue" "queueLength=1" \
  --scale-rule-auth "connection=sec1" \
  --secrets "sec1=<STORAGE_CONNECTION_STRING>" \
  --image alpine:latest \
  --command echo \
  --args "Hello from Queue Job"
```

**Important:** Replace `<STORAGE_CONNECTION_STRING>` with the actual value; don't put the literal placeholder into Azure.

---

# 27. Add a Queue Message Using Azure CLI

## PowerShell

```powershell
az storage message put `
  --queue-name workqueue `
  --content "Task 1" `
  --account-name kmitjobstorage01 `
  --account-key $STORAGE_KEY `
  --auth-mode key
```

## Bash/Linux

```bash
az storage message put \
  --queue-name workqueue \
  --content "Task 1" \
  --account-name kmitjobstorage01 \
  --account-key "$STORAGE_KEY" \
  --auth-mode key
```

For login-based authentication, your Azure identity needs an appropriate Storage Queue data role, such as **Storage Queue Data Contributor**.

---

# 28. Check Job Executions

## PowerShell

```powershell
az containerapp job execution list `
  --name queue-alpine-job `
  --resource-group MYRG-India `
  --output table
```

## Bash/Linux

```bash
az containerapp job execution list \
  --name queue-alpine-job \
  --resource-group MYRG-India \
  --output table
```

Example:

```text
Name                    Status
----------------------  ---------
queue-alpine-job-xxxxx  Succeeded
```

---

# 29. Check the Job Configuration

## PowerShell

```powershell
az containerapp job show `
  --name queue-alpine-job `
  --resource-group MYRG-India `
  --output json
```

## Bash/Linux

```bash
az containerapp job show \
  --name queue-alpine-job \
  --resource-group MYRG-India \
  --output json
```

To specifically inspect the container:

```powershell
az containerapp job show `
  --name queue-alpine-job `
  --resource-group MYRG-India `
  --query "properties.template.containers" `
  --output json
```

---

# 30. Important Difference: Run Now vs KEDA Trigger

### Run Now

```text
User
 ↓
Run Now
 ↓
Job execution
```

This is a **manual execution**.

### KEDA

```text
Queue
 ↓
KEDA
 ↓
Job execution
```

This is an **event-driven execution**.

Therefore, clicking **Run now** is not the same as testing the KEDA trigger.

---

# 31. Complete Learning Flow

The complete learning sequence is:

### Step 1

Create Storage Account.

```text
kmitjobstorage01
```

### Step 2

Create Storage Queue.

```text
workqueue
```

### Step 3

Add test messages.

```text
Task 1
Task 2
Task 3
```

### Step 4

Create Container Apps Job.

```text
queue-alpine-job
```

### Step 5

Select:

```text
Trigger = Event
```

### Step 6

Configure:

```text
Min executions = 0
Max executions = 3
Polling interval = 30
```

### Step 7

Use:

```text
alpine:latest
```

### Step 8

Create secret:

```text
sec1
```

containing the Storage connection information.

### Step 9

Create KEDA scale rule:

```text
Rule name:
queueJob

Type:
azure-queue
```

### Step 10

Configure metadata:

```text
accountName = kmitjobstorage01
queueName   = workqueue
queueLength = 1
```

### Step 11

Configure authentication:

```text
Secret reference:
sec1

Trigger parameter:
connection
```

### Step 12

Test Job execution.

```text
Run Now
```

### Step 13

Understand the result.

The Job succeeds, but:

```text
Task 1
Task 2
Task 3
```

remain in the queue because the Alpine container doesn't process them.

---

# 32. Key Points to Remember

### Point 1

**Event-driven Job** means the Job can be started based on an external event source.

### Point 2

**KEDA** monitors the event source.

### Point 3

For Azure Storage Queue, the KEDA scaler type is:

```text
azure-queue
```

### Point 4

Important metadata:

```text
accountName
queueName
queueLength
```

### Point 5

For connection-string authentication:

```text
Secret reference = your secret
Trigger parameter = connection
```

### Point 6

KEDA **does not consume or delete queue messages**.

### Point 7

The worker application must:

```text
Read
 ↓
Process
 ↓
Delete
```

the queue message.

### Point 8

`Run Now` manually starts an execution. It does not automatically process the Storage Queue.

### Point 9

For a real production-style implementation, use a worker application instead of just:

```text
alpine + echo
```

---

# 33. Final Architecture

The simple demonstration we have completed so far is:

```text
                 Azure
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
 Storage Account       Container Apps
        │                     │
        ▼                     ▼
  workqueue             Event Job
        │                     │
   Task 1, Task 2             │
   Task 3                     │
        │                     │
        └───────┐     ┌───────┘
                ▼     ▼
                 KEDA
                  │
                  ▼
           Start Job Execution
                  │
                  ▼
             Alpine Container
                  │
                  ▼
                echo
                  │
                  ▼
              Succeeded
```

**Important note:** At this stage, we have demonstrated **KEDA triggering an Event-driven Job**, but we have **not yet built the queue-processing worker**. The next logical step is to create a very small Python worker Docker image that actually reads `Task 1`, `Task 2`, etc. from `workqueue`, processes them, deletes them, and exits successfully.
