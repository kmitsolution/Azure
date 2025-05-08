# Workflow of Azure Functions

```plaintext
[Trigger ]
          ↓
[Input Binding ]
          ↓
[Function Logic ]
          ↓
[Output Binding ]
```
---

## 🔄 **Azure Functions Workflow Overview**

### ✅ **1. Trigger**

* This is the **event** that starts the execution of your function.
* Only **one trigger** per function.

### ✅ **2. Input Bindings (optional)**

* These are **external data sources** your function reads from (e.g., Storage, Cosmos DB).
* You can define **multiple input bindings**.

### ✅ **3. Function Logic**

* The **core code** you write (in C#, JavaScript, Python, etc.) that processes the input and performs operations.

### ✅ **4. Output Bindings (optional)**

* These allow the function to **write data to external services** like databases, queues, storage, etc.
* You can have **multiple output bindings**.

---

## 🧠 **Case Study Example**

### 🔄 Scenario: Resize Images on Upload to Azure Blob Storage

#### 🧩 Problem:

You want to automatically resize any image a user uploads to a container in Azure Blob Storage.

#### ✅ Solution: Azure Function

### 🔗 **Workflow**

| Step           | Description                                                |
| -------------- | ---------------------------------------------------------- |
| Trigger        | Blob Storage Trigger: fires when a new image is uploaded   |
| Input Binding  | Reads the uploaded image blob                              |
| Function Logic | Resizes the image using a library (e.g., ImageSharp in C#) |
| Output Binding | Writes the resized image to another blob container         |

#### 📝 Example (C#):

```csharp
[FunctionName("ResizeImage")]
public static async Task Run(
    [BlobTrigger("uploads/{name}", Connection = "AzureWebJobsStorage")] Stream inputBlob,
    [Blob("thumbnails/{name}", FileAccess.Write, Connection = "AzureWebJobsStorage")] Stream outputBlob,
    string name,
    ILogger log)
{
    var image = Image.Load(inputBlob);
    image.Mutate(x => x.Resize(100, 100));
    image.SaveAsJpeg(outputBlob);
}
```

---

## ⏱ **Types of Triggers in Azure Functions**

| Trigger Type            | Description                                                |
| ----------------------- | ---------------------------------------------------------- |
| **HTTP Trigger**        | Executes when an HTTP request is received (REST API style) |
| **Timer Trigger**       | Executes on a schedule (like CRON jobs)                    |
| **Blob Trigger**        | Runs when a new file is added to Blob Storage              |
| **Queue Trigger**       | Runs when a new message appears in a Storage Queue         |
| **Service Bus Trigger** | Listens to Service Bus messages                            |
| **Event Grid Trigger**  | Responds to events from multiple Azure services            |
| **Cosmos DB Trigger**   | Fires when documents change in a Cosmos DB container       |

---

## 📥 **Input Bindings Examples**

| Binding Type      | Example Use                           |
| ----------------- | ------------------------------------- |
| Blob Storage      | Read an image file from storage       |
| Cosmos DB         | Read a document from a NoSQL database |
| HTTP Request Data | Access query parameters, headers      |

---

## 📤 **Output Bindings Examples**

| Binding Type  | Example Use                        |
| ------------- | ---------------------------------- |
| Blob Storage  | Save a file                        |
| Queue Storage | Send a message                     |
| Cosmos DB     | Write a new document               |
| SendGrid      | Send an email                      |
| SignalR       | Send messages to connected clients |

---

## 🛠 **Tools to Create and Deploy Azure Functions**

| Tool                            | Description                                       | Use Case                                                    |
| ------------------------------- | ------------------------------------------------- | ----------------------------------------------------------- |
| **Azure Portal**                | Web-based editor inside Azure                     | Quick edits, testing, and monitoring                        |
| **Visual Studio**               | IDE for .NET and C#                               | Best for C# Azure Functions, full debugging                 |
| **VS Code**                     | Lightweight editor with Azure Functions extension | Best for JavaScript, Python, and multi-platform development |
| **Azure CLI**                   | Command-line tool                                 | Automate deployments, CI/CD pipelines                       |
| **GitHub Actions/Azure DevOps** | CI/CD Tools                                       | Automate builds, tests, and deployments                     |

---

### 🎯 Summary Diagram (Workflow)

```plaintext
[Trigger (e.g., Blob upload)]
          ↓
[Input Binding (Blob data)]
          ↓
[Function Logic (resize image)]
          ↓
[Output Binding (save resized blob)]
```

---


