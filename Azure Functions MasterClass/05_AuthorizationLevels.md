Azure Functions support multiple **authorization levels** to control how clients can access your HTTP-triggered functions. Here's a breakdown of the available options:

---

## 🔐 **Authorization Levels in Azure Functions**

| Level             | Description                                                                                | Use Case                                           | Key Required?   |
| ----------------- | ------------------------------------------------------------------------------------------ | -------------------------------------------------- | --------------- |
| **Anonymous**     | No API key is required. Anyone can call the function.                                      | Public endpoints (e.g., webhooks, open APIs)       | ❌ No            |
| **Function**      | Requires a **function-specific key** (function key) to be passed in the URL.               | For internal services or limited external access   | ✅ Yes           |
| **Admin**         | Requires the **host master key**. Grants full control over all functions.                  | Reserved for admin tasks, deployment scripts       | ✅ Yes           |
| **System (Host)** | Applies to internal system triggers like scale controller (not exposed directly via HTTP). | Used by Azure internally or for host-level actions | 🔒 Internal use |

---

## 🔑 **Types of Keys**

### 🔹 1. **Function Key**

* Specific to a single function.
* Stored and managed under each function.
* URL format:

  ```
  https://<app>.azurewebsites.net/api/<function>?code=<function-key>
  ```

### 🔹 2. **Host Key**

* Shared across all functions within the Function App.
* Useful when multiple functions need the same access control.
* Found under: **Function App → Functions → Host Keys**

### 🔹 3. **Admin Key (Master Key)**

* Highest privilege level.
* Grants access to all functions and administrative APIs.
* Should be used cautiously and stored securely.
* Can be found in: **Function App → App Keys → *master***

---

## 🌐 Example: Calling Functions with Keys

```http
GET https://<yourapp>.azurewebsites.net/api/HttpTrigger1?code=<your-key>
```

* If `AuthorizationLevel` is:

  * `Anonymous` → no key needed.
  * `Function` → must supply function key or host key.
  * `Admin` → must supply the master key.

---

## 📌 Recommendation

| Scenario                            | Recommended Level |
| ----------------------------------- | ----------------- |
| Public APIs / Webhooks              | `Anonymous`       |
| Internal systems / limited partners | `Function`        |
| Deployment tools / automation       | `Admin`           |

---


