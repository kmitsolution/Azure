Yes. Now we move from **“KEDA detects the queue”** to **“the container actually reads and processes the queue message.”**

Microsoft's event-driven Job tutorial follows exactly this pattern: each execution gets a message, processes it, deletes it, and exits. KEDA only monitors queue length and starts executions; it doesn't process the message itself. ([Microsoft Learn][1])

## 1. Our target architecture

We currently have:

```text
Storage Account
     │
     ▼
 workqueue
 ├── Task 1
 ├── Task 2
 └── Task 3
     │
     ▼
    KEDA
     │
     ▼
Container Apps Job
     │
     ▼
Python Worker
     │
     ├── Read message
     ├── Process message
     └── Delete message
```

We'll create a very simple **Python worker**.

---

# 2. Create the worker application

Create a directory:

```text
queue-worker
```

Inside it create:

```text
queue-worker/
│
├── worker.py
├── requirements.txt
└── Dockerfile
```

---

# 3. Create `requirements.txt`

```text
azure-storage-queue
```

The official Azure Python Queue Storage library supports receiving and deleting queue messages. ([Microsoft Learn][2])

---

# 4. Create `worker.py`

Use this simple code:

```python
import os
import time
from azure.storage.queue import QueueClient

connection_string = os.environ["AZURE_STORAGE_CONNECTION_STRING"]
queue_name = os.environ["AZURE_STORAGE_QUEUE_NAME"]

queue_client = QueueClient.from_connection_string(
    conn_str=connection_string,
    queue_name=queue_name
)

print("Queue worker started")
print(f"Queue: {queue_name}")

messages = queue_client.receive_messages(
    messages_per_page=1,
    visibility_timeout=60
)

processed = False

for message in messages:
    print(f"Received message: {message.content}")

    # Simulate processing
    print("Processing message...")
    time.sleep(5)

    print(f"Task completed: {message.content}")

    # Delete only after successful processing
    queue_client.delete_message(message)

    print("Message deleted from queue")

    processed = True
    break

if not processed:
    print("No message found")

print("Worker completed")
```

---

# 5. Understand the important part

The critical flow is:

```text
receive_messages()
       ↓
Read Task 1
       ↓
Process Task 1
       ↓
delete_message()
       ↓
Task 1 removed
```

This is what our Alpine container was missing.

Previously:

```text
Alpine
   ↓
echo
   ↓
exit
```

Now:

```text
Python Worker
   ↓
Connect to Azure Queue
   ↓
Get message
   ↓
Process message
   ↓
Delete message
   ↓
exit
```

---

# 6. Create the Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY worker.py .

CMD ["python", "worker.py"]
```

---

# 7. Build the Docker image

For the first test, you need to put this image in a container registry accessible by Container Apps.

Since you already have Azure Container Registry experience, let's use ACR.

For example:

```text
ACR:
kmitacr01
```

Login:

### PowerShell

```powershell
az acr login --name kmitacr01
```

### Bash

```bash
az acr login --name kmitacr01
```

Build:

### PowerShell

```powershell
docker build -t kmitacr01.azurecr.io/queue-worker:v1 .
```

### Bash

```bash
docker build -t kmitacr01.azurecr.io/queue-worker:v1 .
```

Push:

### PowerShell

```powershell
docker push kmitacr01.azurecr.io/queue-worker:v1
```

### Bash

```bash
docker push kmitacr01.azurecr.io/queue-worker:v1
```

---

# 8. Configure the Container Apps Job

Now our Job needs two environment variables.

```text
AZURE_STORAGE_CONNECTION_STRING
AZURE_STORAGE_QUEUE_NAME
```

The queue name is not sensitive:

```text
AZURE_STORAGE_QUEUE_NAME=workqueue
```

The connection string **is sensitive**, so reference the Job secret.

Conceptually:

```text
Secret
  │
  │ sec1
  ▼
Environment variable
  │
  │ AZURE_STORAGE_CONNECTION_STRING
  ▼
Python Worker
```

---

# 9. Portal configuration

Open:

**Container Apps Jobs → queue-alpine-job**

Go to:

**Containers**

Edit the container.

Change the image from:

```text
alpine:latest
```

to:

```text
kmitacr01.azurecr.io/queue-worker:v1
```

Then open:

**Environment variables**

Add:

### Variable 1

```text
Name:
AZURE_STORAGE_CONNECTION_STRING
```

For the value, select:

```text
Secret reference
```

and select:

```text
sec1
```

### Variable 2

```text
Name:
AZURE_STORAGE_QUEUE_NAME

Value:
workqueue
```

You should have:

```text
Environment variables
──────────────────────────────────────

AZURE_STORAGE_CONNECTION_STRING
    Secret reference → sec1

AZURE_STORAGE_QUEUE_NAME
    workqueue
```

The Microsoft event-driven Job tutorial uses the same pattern: the Storage connection string is kept as a secret and exposed to the worker through an environment variable. ([Microsoft Learn][1])

---

# 10. Keep the KEDA scale rule

Don't remove your existing scale rule.

You already have:

```text
Rule name:
queueJob

Type:
azure-queue
```

Metadata:

```text
accountName = kmitjobstorage01
queueName   = workqueue
queueLength = 1
```

Authentication:

```text
Secret reference:
sec1

Trigger parameter:
connection
```

So there are actually **two uses of `sec1`**:

```text
                  sec1
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
      KEDA              Python Worker
        │                     │
        │                     │
   Monitor queue       Connect to queue
```

This is an important distinction.

---

# 11. What happens when we add Task 1?

Suppose:

```text
workqueue
────────────
Task 1
```

KEDA polls the queue.

```text
KEDA
  │
  │ Queue length = 1
  ▼
Start Job execution
```

The Container Apps Job starts:

```text
queue-worker:v1
```

The Python program executes:

```python
messages = queue_client.receive_messages(...)
```

It receives:

```text
Task 1
```

Then:

```text
Processing Task 1
```

After processing:

```python
queue_client.delete_message(message)
```

Task 1 is deleted.

Final queue:

```text
workqueue
────────────

EMPTY
```

---

# 12. Very important: delete after processing

Don't do this:

```python
receive message
    ↓
delete message
    ↓
process message
```

Instead:

```text
Receive
   ↓
Process
   ↓
Successful?
   │
   ▼
Delete
   ↓
Exit
```

Why?

Suppose your application crashes while processing:

```text
Task 1
```

If you've already deleted it, the task can be lost.

The Azure Container Apps event-driven Job tutorial specifically notes that messages shouldn't be deleted until processing is finished, because KEDA uses queue length for scaling. ([Microsoft Learn][1])

---

# 13. Test with three messages

Add:

```text
Task 1
Task 2
Task 3
```

You can use Portal or CLI.

### PowerShell

```powershell
az storage message put `
  --queue-name workqueue `
  --content "Task 1" `
  --account-name kmitjobstorage01 `
  --account-key $STORAGE_KEY `
  --auth-mode key
```

Then:

```powershell
az storage message put `
  --queue-name workqueue `
  --content "Task 2" `
  --account-name kmitjobstorage01 `
  --account-key $STORAGE_KEY `
  --auth-mode key
```

And:

```powershell
az storage message put `
  --queue-name workqueue `
  --content "Task 3" `
  --account-name kmitjobstorage01 `
  --account-key $STORAGE_KEY `
  --auth-mode key
```

---

# 14. Watch the Job executions

Run:

```powershell
az containerapp job execution list `
  --name queue-alpine-job `
  --resource-group MYRG-India `
  --output table
```

You should start seeing executions.

For example:

```text
Name                         Status
---------------------------  ---------
queue-alpine-job-abc123      Succeeded
queue-alpine-job-def456      Succeeded
queue-alpine-job-ghi789      Succeeded
```

Depending on the queue/scaling timing, the exact execution pattern can vary.

---

# 15. Check the logs

Now the logs become very useful.

Use:

```powershell
az containerapp job logs show `
  --name queue-alpine-job `
  --resource-group MYRG-India `
  --execution <EXECUTION_NAME> `
  --container <CONTAINER_NAME>
```

For example:

```powershell
az containerapp job logs show `
  --name queue-alpine-job `
  --resource-group MYRG-India `
  --execution queue-alpine-job-abc123 `
  --container queue-worker
```

You should see:

```text
Queue worker started
Queue: workqueue
Received message: Task 1
Processing message...
Task completed: Task 1
Message deleted from queue
Worker completed
```

---

# 16. Verify the Queue

Go back to:

**Storage Account → Queues → workqueue**

Initially:

```text
Task 1
Task 2
Task 3
```

After processing:

```text
EMPTY
```

That's the complete event-driven workflow.

---

# 17. One important improvement to our worker

There is one subtle issue with the simple demonstration code.

We're using:

```python
receive_messages(
    messages_per_page=1,
    visibility_timeout=60
)
```

and processing one message.

That's intentional for learning.

It gives us a very easy model:

```text
1 Job execution
       ↓
1 message
       ↓
process
       ↓
delete
       ↓
exit
```

This matches the conceptual model Microsoft uses for its event-driven Job tutorial: each execution processes a queue message and then stops. ([Microsoft Learn][1])

---

# 18. Final Architecture

Now our architecture becomes:

```text
                    Azure Storage
                         │
                         ▼
                    workqueue
                 ┌───────────────┐
                 │ Task 1        │
                 │ Task 2        │
                 │ Task 3        │
                 └───────┬───────┘
                         │
                         ▼
                       KEDA
                         │
                  queueLength = 1
                         │
                         ▼
              Container Apps Job
                         │
                         ▼
                  queue-worker
                         │
                ┌────────┴────────┐
                │                 │
                ▼                 ▼
          Receive message     Process task
                │                 │
                └────────┬────────┘
                         ▼
                  Delete message
                         │
                         ▼
                    Job exits
```

### The key concept

**KEDA decides when to start the Job.**

**Your application decides what to do with the message.**

That distinction is the main thing to understand for **Event-driven Jobs with KEDA**. ([Microsoft Learn][3])

If you want to continue this lab, the next useful step is to make the **Python worker process one queue message per execution with proper retry/failure behavior**, so we can demonstrate what happens when `Task 2` fails and how the queue message becomes available again.

[1]: https://learn.microsoft.com/nb-no/azure/container-apps/tutorial-event-driven-jobs?utm_source=chatgpt.com "Tutorial: Deploy an Event-Driven Job with Azure Container Apps | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/azure/storage/queues/storage-quickstart-queues-python?utm_source=chatgpt.com "Quickstart: Azure Queue Storage client library for Python - Azure Storage | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/azure/container-apps/jobs?utm_source=chatgpt.com "Jobs in Azure Container Apps | Microsoft Learn"
