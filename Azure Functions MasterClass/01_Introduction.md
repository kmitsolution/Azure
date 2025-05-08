### **Serverless Computing**

**Serverless computing** is a cloud computing model where the cloud provider automatically manages the infrastructure, scaling, and server provisioning. Despite the name, it doesn’t mean there are no servers—it means developers don’t need to manage them.

#### Key characteristics:

* **No server management:** The cloud provider handles all server maintenance.
* **Event-driven execution:** Code runs in response to events or triggers.
* **Auto-scaling:** Automatically scales with demand.
* **Pay-per-execution:** You are charged only when your code runs.

---

### **Azure Functions: Overview** <img width="50" alt="image" src="https://github.com/user-attachments/assets/b24b65a2-df5e-48fd-91ec-9f5f41605788" />

**Azure Functions** is Microsoft Azure's serverless compute service. It lets you run small pieces of code (functions) without worrying about the application infrastructure.

#### Features:

* **Trigger-based execution:** Supports triggers like HTTP requests, timers, blob storage changes, queue messages, etc.
* **Multiple languages:** Supports C#, JavaScript, Python, Java, PowerShell, and more.
* **Integrated with Azure services:** Works seamlessly with services like Cosmos DB, Event Grid, Service Bus.
* **Consumption plan (default):** Automatically scales and you pay only for execution time and resources used.

#### Common use cases:

* Processing data from IoT devices.
* Scheduled data cleanup or processing.
* Backend logic for web APIs or mobile apps.
* Event-driven integrations (e.g., file uploaded to storage).

---

<img width="515" alt="image" src="https://github.com/user-attachments/assets/f33da6e1-0b5f-4c58-a1e6-cf8f2759e6d1" />

### **Azure Functions vs Azure App Services**

| Feature             | **Azure Functions**                        | **Azure App Service**                      |
| ------------------- | ------------------------------------------ | ------------------------------------------ |
| **Architecture**    | Serverless                                 | PaaS (Platform as a Service)               |
| **Execution Model** | Event-driven, short-lived                  | Always-on, suitable for long-running apps  |
| **Scaling**         | Automatic (event-based)                    | Manual or auto-scale based on rules        |
| **Billing**         | Pay-per-use (consumption)                  | Pay for reserved compute (even when idle)  |
| **Use Case**        | Lightweight, microservice, background jobs | Web apps, APIs, and mobile app backends    |
| **Startup Time**    | May have cold starts                       | Always-on (no cold start)                  |
| **State**           | Stateless                                  | Can be stateful (via session storage etc.) |

---

### Summary:

* Use **Azure Functions** for lightweight, background tasks, or microservices that respond to events and scale automatically.
* Use **Azure App Service** when you need a full-fledged web app or API that runs continuously with more control over the hosting environment.


