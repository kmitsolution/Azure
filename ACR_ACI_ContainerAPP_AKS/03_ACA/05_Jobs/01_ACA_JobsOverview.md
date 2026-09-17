# Azure Container Apps Jobs — Introduction

Azure Container Apps Jobs are designed for **containerized workloads that start, perform a task, and then finish**.

A normal Azure Container App is generally designed for a **long-running application**, while an ACA Job is designed for a **run-to-completion task**.

---

## 1. What is an Azure Container Apps Job?

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

This is the first concept to understand.

| Feature           | Container App            | Container Apps Job     |
| ----------------- | ------------------------ | ---------------------- |
| Main purpose      | Long-running application | Run-to-completion task |
| Typical workload  | Web/API                  | Batch/background task  |
| HTTP endpoint     | Common                   | Usually not required   |
| Runs continuously | Usually                  | No                     |
| Finishes          | Usually doesn't          | Yes                    |
| Trigger           | HTTP/events              | Manual/schedule/events |
| Example           | Nginx, Node.js API       | Backup script          |
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

Imagine you have an e-commerce application.

The application is:

```text
Frontend
   ↓
Backend API
   ↓
Database
```

That's a normal Container App.

But every night you need to generate a sales report:

```text
12:00 AM
   ↓
Start container
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

Suppose you have 1 million customer records and want to process them.

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

You explicitly start the job.

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

The job runs according to a **cron expression**.

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

The job starts when an event occurs.

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

# 9. First Simple Demo

For our first demo, let's use a very small container image.

We can use:

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

# 10. Verify the Container Apps Environment

For our lab, let's use:

```text
Resource Group:
MYRG-India

Container Apps Environment:
con-env
```

### Bash / Linux / Azure CLI

```bash
az containerapp env show \
  --name con-env \
  --resource-group MYRG-India \
  --output table
```

If the environment exists, Azure will display its details.

---

# 11. Create a Manual ACA Job

### Bash / Linux / Azure CLI

```bash
az containerapp job create \
  --name hello-job \
  --resource-group MYRG-India \
  --environment con-env \
  --trigger-type Manual \
  --image mcr.microsoft.com/k8se/quickstart:latest \
  --cpu 0.25 \
  --memory 0.5Gi
```

The important parameters are:

```text
--name
```

Name of the Job.

```text
--resource-group
```

Resource group containing the Job.

```text
--environment
```

Container Apps Environment where the Job runs.

```text
--trigger-type Manual
```

The Job runs when we explicitly start it.

```text
--image
```

Container image to run.

```text
--cpu
```

CPU allocated to the container.

```text
--memory
```

Memory allocated to the container.

---

# 12. Start the Job

After creating the Job:

### Bash / Linux / Azure CLI

```bash
az containerapp job start \
  --name hello-job \
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

# 13. List Job Executions

After starting the Job:

### Bash / Linux / Azure CLI

```bash
az containerapp job execution list \
  --name hello-job \
  --resource-group MYRG-India \
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

# 14. View the Job

You can check the Job configuration with:

### Bash / Linux / Azure CLI

```bash
az containerapp job show \
  --name hello-job \
  --resource-group MYRG-India \
  --output table
```

For detailed JSON:

```bash
az containerapp job show \
  --name hello-job \
  --resource-group MYRG-India \
  --output json
```

---

# 15. Manual Job Flow

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

# 16. Where ACA Jobs Fit with Container Apps

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

# 17. Basic Azure CLI Commands to Remember

### Create Job

```bash
az containerapp job create \
  --name hello-job \
  --resource-group MYRG-India \
  --environment con-env \
  --trigger-type Manual \
  --image mcr.microsoft.com/k8se/quickstart:latest \
  --cpu 0.25 \
  --memory 0.5Gi
```

### Start Job

```bash
az containerapp job start \
  --name hello-job \
  --resource-group MYRG-India
```

### List Executions

```bash
az containerapp job execution list \
  --name hello-job \
  --resource-group MYRG-India \
  --output table
```

### Show Job

```bash
az containerapp job show \
  --name hello-job \
  --resource-group MYRG-India \
  --output json
```

### Delete Job

```bash
az containerapp job delete \
  --name hello-job \
  --resource-group MYRG-India \
  --yes
```

---

# 18. Important Terminology

Keep these four concepts separate:

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

# 19. Learning Path for ACA Jobs

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
05. Replicas in Jobs
        ↓
06. Job Restart / Retry
        ↓
07. Environment Variables
        ↓
08. Secrets
        ↓
09. Job Scaling
        ↓
10. KEDA + Event-driven Jobs
        ↓
11. Cron Expressions
        ↓
12. YAML Manifest
        ↓
13. Azure CLI
        ↓
14. Portal
        ↓
15. Real-world examples
```

## Important Note

The most important difference to remember is:

> **A Container App is primarily for continuously serving an application, whereas an Azure Container Apps Job is for running a task to completion.**

For the next practical lesson, the natural step is **ACA Job Types — Manual vs Scheduled vs Event-driven**, where we can create each type using **Azure CLI with Bash commands**.
