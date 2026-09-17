# Azure Container Apps Jobs — Introduction

Azure Container Apps Jobs are designed for **containerized workloads that start, perform a task, and then finish**.

A normal Azure Container App is generally designed for a **long-running application**, while an ACA Job is designed for a **run-to-completion task**.

---

# 1. What is an Azure Container Apps Job?

A normal Container App typically works like this:

```text
User
  ↓
Container App
  ↓
Web Application / API
  ↓
Keeps running
```

An ACA Job works like this:

```text
Trigger
   ↓
ACA Job
   ↓
Container starts
   ↓
Task executes
   ↓
Task completes
   ↓
Container stops
```

### Examples

ACA Jobs are useful for:

* Database migration
* Batch processing
* Image/video processing
* Sending a batch of emails
* Data import/export
* Scheduled scripts
* Background processing
* ETL workloads
* Report generation
* Queue-based processing

---

# 2. Container App vs Container Apps Job

| Feature           | Container App            | Container Apps Job     |
| ----------------- | ------------------------ | ---------------------- |
| Main purpose      | Long-running application | Run-to-completion task |
| Typical workload  | Web/API                  | Batch/background task  |
| HTTP endpoint     | Common                   | Usually not required   |
| Runs continuously | Usually                  | No                     |
| Finishes          | Usually doesn't          | Yes                    |
| Trigger           | HTTP/events              | Manual/schedule/events |
| Example           | Nginx, Node.js API       | Database migration     |
| Scaling           | Application workload     | Job executions         |

### Simple memory trick

```text
Container App
     ↓
"Keep serving"

Container Apps Job
     ↓
"Do the work and finish"
```

---

# 3. Simple Real-World Example

Imagine you have an e-commerce application:

```text
Frontend
   ↓
Backend API
   ↓
Database
```

The frontend and backend are long-running applications, so they can run as Container Apps.

But every night you need to generate a sales report:

```text
12:00 AM
   ↓
Start Job
   ↓
Read database
   ↓
Generate report
   ↓
Save report
   ↓
Finish
   ↓
Container stops
```

This is a good use case for an **ACA Job**.

---

# 4. Another Simple Example

Suppose you have 1 million customer records to process.

Instead of keeping a container running all the time:

```text
Container
   ↓
Always running
   ↓
Waiting for work
```

you can use a Job:

```text
ACA Job
   ↓
Start
   ↓
Process records
   ↓
Complete
   ↓
Stop
```

---

# 5. ACA Job Architecture

A simple architecture looks like this:

```text
              Azure Container Apps Environment
                           │
              ┌────────────┴────────────┐
              │                         │
        Container App            Container Apps Job
              │                         │
              │                         │
          Web / API                  Batch Task
                                        │
                                        ▼
                                      Start
                                        │
                                     Process
                                        │
                                     Complete
                                        │
                                      Stop
```

An ACA Job runs inside a **Container Apps Environment**, just like a normal Container App.

---

# 6. Three Types of ACA Jobs

Azure Container Apps Jobs have three trigger types:

```text
                    ACA Jobs
                       │
          ┌────────────┼────────────┐
          │            │            │
        Manual       Schedule      Event
          │            │            │
          ▼            ▼            ▼
        Run it        Cron        KEDA/Event
       yourself     schedule       driven
```

---

## 6.1 Manual Job

You explicitly start the Job.

```text
User
 ↓
Start Job
 ↓
Container runs
 ↓
Complete
```

Useful for:

* Testing
* One-time processing
* Database migration
* Administrative tasks

---

## 6.2 Scheduled Job

The Job runs according to a **cron expression**.

Example:

```text
Every day at midnight
```

```text
00:00
  ↓
Job
  ↓
Process
  ↓
Complete

Next day

00:00
  ↓
Job
  ↓
Process
  ↓
Complete
```

Useful for:

* Daily reports
* Database cleanup
* Backups
* Data synchronization
* Nightly processing

---

## 6.3 Event-Driven Job

The Job starts when an event occurs.

For example:

```text
Queue
  ↓
Messages arrive
  ↓
KEDA detects messages
  ↓
ACA Job starts
  ↓
Process messages
  ↓
Complete
```

This is where **KEDA** becomes important.

---

# 7. What is a Job Execution?

A **Job** is the definition/configuration.

An **Execution** is one actual run of that Job.

For example:

```text
Job:
daily-report
```

You run it:

```text
Execution 1
```

Tomorrow:

```text
Execution 2
```

Next day:

```text
Execution 3
```

So:

```text
             ACA Job
                │
       ┌────────┼────────┐
       │        │        │
   Execution 1 Execution 2 Execution 3
       │        │        │
      Done     Done     Done
```

---

# 8. Job vs Execution vs Replica

These three terms can initially be confusing.

### Job

Defines **what should run**.

```text
daily-report
```

### Execution

One actual run of the Job.

```text
daily-report
     ↓
Execution 1
```

### Replica

The actual container instance doing the work.

```text
Execution 1
     ↓
Replica
     ↓
Container
```

For example:

```text
Job
 │
 └── Execution
       │
       ├── Replica 1
       ├── Replica 2
       └── Replica 3
```

This becomes useful when a Job needs to process a large amount of work in parallel.

---

# 9. Replica Timeout and Retry Limit

Two important settings for an ACA Job are:

```text
--replica-timeout
--replica-retry-limit
```

These control what happens to an individual Job replica when the task takes too long or fails.

---

## 9.1 `--replica-timeout`

Example:

```text
--replica-timeout 60
```

This means the Job replica is allowed to run for up to **60 seconds**.

Conceptually:

```text
Replica starts
     ↓
Application runs
     ↓
60 seconds
     ↓
If still running
     ↓
Replica times out
```

So:

```text
--replica-timeout 60
```

is useful for preventing a Job from running indefinitely.

---

## 9.2 `--replica-retry-limit`

Example:

```text
--replica-retry-limit 2
```

This controls retry behavior when a replica fails.

Conceptually:

```text
Replica starts
     ↓
Fails
     ↓
Retry
     ↓
Fails
     ↓
Retry
     ↓
Fails
     ↓
Execution eventually fails
```

### Simple memory trick

```text
Timeout
   ↓
"How long can it run?"

Retry Limit
   ↓
"How many times can it retry?"
```

---

# 10. First Simple Demo

For our first demo, let's use a very small container image:

```text
mcr.microsoft.com/k8se/quickstart:latest
```

The idea is:

```text
ACA Job
   ↓
Container
   ↓
Run task
   ↓
Complete
```

There is no complicated application to build.

---

# 11. Verify the Container Apps Environment

For our lab:

```text
Resource Group:
MYRG-India

Container Apps Environment:
con-env
```

## Bash / Linux / Azure CLI

```bash
az containerapp env show \
  --name con-env \
  --resource-group MYRG-India \
  --output table
```

## PowerShell / Azure CLI

```powershell
az containerapp env show `
  --name con-env `
  --resource-group MYRG-India `
  --output table
```

---

# 12. Create a Manual ACA Job

For the first demo, we'll create a **Manual Job**.

## Bash / Linux / Azure CLI

```bash
az containerapp job create \
  --name hello-job \
  --resource-group MYRG-India \
  --environment con-env \
  --trigger-type Manual \
  --image mcr.microsoft.com/k8se/quickstart:latest \
  --cpu 0.25 \
  --memory 0.5Gi \
  --replica-timeout 60 \
  --replica-retry-limit 2
```

## PowerShell / Azure CLI

```powershell
az containerapp job create `
  --name hello-job `
  --resource-group MYRG-India `
  --environment con-env `
  --trigger-type Manual `
  --image mcr.microsoft.com/k8se/quickstart:latest `
  --cpu 0.25 `
  --memory 0.5Gi `
  --replica-timeout 60 `
  --replica-retry-limit 2
```

### Important parameters

| Parameter                 | Meaning                               |
| ------------------------- | ------------------------------------- |
| `--name`                  | Name of the Job                       |
| `--resource-group`        | Resource group containing the Job     |
| `--environment`           | Container Apps Environment            |
| `--trigger-type Manual`   | Job starts manually                   |
| `--image`                 | Container image                       |
| `--cpu`                   | CPU allocated to the container        |
| `--memory`                | Memory allocated to the container     |
| `--replica-timeout 60`    | Maximum runtime allowed for a replica |
| `--replica-retry-limit 2` | Retry limit for a failed replica      |

---

# 13. Start the Job

After creating the Job:

## Bash / Linux / Azure CLI

```bash
az containerapp job start \
  --name hello-job \
  --resource-group MYRG-India
```

## PowerShell / Azure CLI

```powershell
az containerapp job start `
  --name hello-job `
  --resource-group MYRG-India
```

The flow is:

```text
hello-job
    ↓
Start
    ↓
Execution
    ↓
Container
    ↓
Task
    ↓
Completed
```

---

# 14. List Job Executions

After starting the Job:

## Bash / Linux / Azure CLI

```bash
az containerapp job execution list \
  --name hello-job \
  --resource-group MYRG-India \
  --output table
```

## PowerShell / Azure CLI

```powershell
az containerapp job execution list `
  --name hello-job `
  --resource-group MYRG-India `
  --output table
```

You may see something similar to:

```text
Name                  Status
--------------------  ---------
hello-job-xxxxx       Succeeded
```

The exact execution name is generated by Azure.

---

# 15. View the Job

## Bash / Linux / Azure CLI

```bash
az containerapp job show \
  --name hello-job \
  --resource-group MYRG-India \
  --output json
```

## PowerShell / Azure CLI

```powershell
az containerapp job show `
  --name hello-job `
  --resource-group MYRG-India `
  --output json
```

For a table:

### Bash

```bash
az containerapp job show \
  --name hello-job \
  --resource-group MYRG-India \
  --output table
```

### PowerShell

```powershell
az containerapp job show `
  --name hello-job `
  --resource-group MYRG-India `
  --output table
```

---

# 16. Manual Job Flow

The complete flow is:

```text
             MANUAL JOB

                User
                  │
                  ▼
             Start Job
                  │
                  ▼
             Job Execution
                  │
                  ▼
               Container
                  │
                  ▼
              Do Work
                  │
                  ▼
               Finish
                  │
                  ▼
              Succeeded
```

Unlike a normal Container App:

```text
Browser
   ↓
Container App
   ↓
Nginx
   ↓
Keep running
```

The Job is:

```text
Trigger
   ↓
Container
   ↓
Work
   ↓
Exit
```

---

# 17. Understanding Timeout and Retry Together

Consider:

```text
--replica-timeout 60
--replica-retry-limit 2
```

Suppose your application gets stuck:

```text
Replica starts
     ↓
Application runs
     ↓
60 seconds
     ↓
Timeout
```

If the replica fails and retries are configured:

```text
Attempt
   ↓
Failure
   ↓
Retry
   ↓
Failure
   ↓
Retry
   ↓
Failure
```

The retry behavior is controlled by the Job's retry configuration.

### Simple memory trick

```text
Timeout
   ↓
"How long can it run?"

Retry Limit
   ↓
"How many retry attempts are allowed?"
```

---

# 18. Where ACA Jobs Fit with Container Apps

We have:

```text
              Container Apps Environment
                       │
             ┌─────────┴─────────┐
             │                   │
      Container App        Container Apps Job
             │                   │
        Long-running        Run-to-completion
             │                   │
        Web/API/App         Batch/Background
```

### Container App

Use it for:

```text
Web application
REST API
Backend service
Microservice
```

Example:

```text
Frontend → Backend API
```

### ACA Job

Use it for:

```text
Batch processing
Scheduled task
Data processing
Migration
One-time task
Background processing
Queue-driven workload
```

Example:

```text
Database
   ↓
ACA Job
   ↓
Generate report
   ↓
Storage
```

---

# 19. Basic Azure CLI Commands to Remember

## Create Job

### Bash / Linux

```bash
az containerapp job create \
  --name hello-job \
  --resource-group MYRG-India \
  --environment con-env \
  --trigger-type Manual \
  --image mcr.microsoft.com/k8se/quickstart:latest \
  --cpu 0.25 \
  --memory 0.5Gi \
  --replica-timeout 60 \
  --replica-retry-limit 2
```

### PowerShell

```powershell
az containerapp job create `
  --name hello-job `
  --resource-group MYRG-India `
  --environment con-env `
  --trigger-type Manual `
  --image mcr.microsoft.com/k8se/quickstart:latest `
  --cpu 0.25 `
  --memory 0.5Gi `
  --replica-timeout 60 `
  --replica-retry-limit 2
```

---

## Start Job

### Bash / Linux

```bash
az containerapp job start \
  --name hello-job \
  --resource-group MYRG-India
```

### PowerShell

```powershell
az containerapp job start `
  --name hello-job `
  --resource-group MYRG-India
```

---

## List Executions

### Bash / Linux

```bash
az containerapp job execution list \
  --name hello-job \
  --resource-group MYRG-India \
  --output table
```

### PowerShell

```powershell
az containerapp job execution list `
  --name hello-job `
  --resource-group MYRG-India `
  --output table
```

---

## Show Job

### Bash / Linux

```bash
az containerapp job show \
  --name hello-job \
  --resource-group MYRG-India \
  --output json
```

### PowerShell

```powershell
az containerapp job show `
  --name hello-job `
  --resource-group MYRG-India `
  --output json
```

---

## Delete Job

### Bash / Linux

```bash
az containerapp job delete \
  --name hello-job \
  --resource-group MYRG-India \
  --yes
```

### PowerShell

```powershell
az containerapp job delete `
  --name hello-job `
  --resource-group MYRG-India `
  --yes
```

---

# 20. Important Terminology

Keep these concepts separate:

```text
Environment
     ↓
Where the Job runs

Job
     ↓
Defines what should run

Execution
     ↓
One actual run of the Job

Replica
     ↓
Actual container instance running the work

Replica Timeout
     ↓
Maximum runtime allowed for a replica

Replica Retry Limit
     ↓
Retry behavior after replica failure
```

For example:

```text
con-env
   │
   └── hello-job
          │
          └── Execution 1
                 │
                 └── Replica
                       │
                       └── Container
```

---

# 21. Learning Path for ACA Jobs

We can cover ACA Jobs in this order:

```text
01. ACA Jobs Introduction
        ↓
02. Container App vs Job
        ↓
03. Job Types
    ├── Manual
    ├── Schedule
    └── Event
        ↓
04. Job Execution
        ↓
05. Replica
        ↓
06. Replica Timeout
        ↓
07. Replica Retry Limit
        ↓
08. Environment Variables
        ↓
09. Secrets
        ↓
10. Job Scaling
        ↓
11. KEDA + Event-driven Jobs
        ↓
12. Cron Expressions
        ↓
13. YAML Manifest
        ↓
14. Azure CLI
        ↓
15. Portal
        ↓
16. Real-world examples
```

## Important Note

The most important difference to remember:

> **A Container App is primarily for continuously serving an application, whereas an Azure Container Apps Job is for running a task to completion.**

For Job execution:

> **`--replica-timeout` controls how long a replica is allowed to run, while `--replica-retry-limit` controls retry behavior when a replica fails.**

Yes — your observation is correct. The problem is not the `replica-timeout`. The problem is that **Alpine's default behavior is being used in a way that keeps the container alive**, so ACA eventually hits the 60-second deadline.

Let's use an even simpler approach: **use the Alpine image with its command explicitly set to `echo`**, without `/bin/sh`, `-c`, or multiple arguments.

## 1. Delete the existing Job

### PowerShell

```powershell
az containerapp job delete `
  --name alpine-job `
  --resource-group MYRG-India `
  --yes
```

### Bash

```bash
az containerapp job delete \
  --name alpine-job \
  --resource-group MYRG-India \
  --yes
```

---

# 2. Create the Job

We'll make the container execute:

```text
echo Hello from ACA Job
```

The `echo` process finishes immediately with exit code `0`.

### PowerShell

```powershell
az containerapp job create `
  --name alpine-job `
  --resource-group MYRG-India `
  --environment con-env `
  --trigger-type Manual `
  --image alpine:latest `
  --command echo `
  --args "Hello from ACA Job" `
  --cpu 0.25 `
  --memory 0.5Gi `
  --replica-timeout 60 `
  --replica-retry-limit 2
```

### Bash

```bash
az containerapp job create \
  --name alpine-job \
  --resource-group MYRG-India \
  --environment con-env \
  --trigger-type Manual \
  --image alpine:latest \
  --command echo \
  --args "Hello from ACA Job" \
  --cpu 0.25 \
  --memory 0.5Gi \
  --replica-timeout 60 \
  --replica-retry-limit 2
```

The important part is:

```text
--command echo
--args "Hello from ACA Job"
```

There is **no shell involved**.

The container should do:

```text
Alpine container starts
        ↓
echo Hello from ACA Job
        ↓
Output generated
        ↓
echo exits
        ↓
Container stops
        ↓
Execution = Succeeded
```

---

# 3. Start the Job

### PowerShell

```powershell
az containerapp job start `
  --name alpine-job `
  --resource-group MYRG-India
```

### Bash

```bash
az containerapp job start \
  --name alpine-job \
  --resource-group MYRG-India
```

---

# 4. Check the Execution

### PowerShell

```powershell
az containerapp job execution list `
  --name alpine-job `
  --resource-group MYRG-India `
  --output table
```

### Bash

```bash
az containerapp job execution list \
  --name alpine-job \
  --resource-group MYRG-India \
  --output table
```

You should see:

```text
Name                 StartTime                  Status
-------------------  -------------------------  ---------
alpine-job-xxxxxxx   2026-09-17T...             Succeeded
```

---

# 5. Before Starting, Verify the Command

This is particularly useful in your lab.

### PowerShell

```powershell
az containerapp job show `
  --name alpine-job `
  --resource-group MYRG-India `
  --query "properties.template.containers[0]" `
  --output json
```

You should see something conceptually like:

```json
{
  "name": "alpine",
  "image": "alpine:latest",
  "command": [
    "echo"
  ],
  "args": [
    "Hello from ACA Job"
  ]
}
```

This confirms that Azure isn't simply starting the Alpine container with its default command.

---

## Important Concept

Your previous configuration effectively allowed the container to remain alive:

```text
Container starts
      ↓
Container keeps running
      ↓
60 seconds
      ↓
Replica timeout
      ↓
Failed
```

The new configuration is:

```text
Container starts
      ↓
echo runs
      ↓
echo exits
      ↓
Container exits
      ↓
ACA sees exit code 0
      ↓
Succeeded
```

