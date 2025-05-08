**Step-by-Step documented guide** to **create an Azure Function App** using the **Azure Portal**, based on your instructions:

---

# 📄 **How to Create an Azure Function App in Azure Portal**

Follow these steps to create and configure your Azure Function App:

---

### ✅ **Step 1: Navigate to Function App**

1. Sign in to the [Azure Portal](https://portal.azure.com).
2. In the search bar, type **"Function App"**.
3. Click **“Create”** to begin setting up a new Function App.

---

### ✅ **Step 2: Configure the Basics**

1. **Select Plan Type**:

   * Choose **Consumption Plan** (or another plan depending on your needs).

2. **Select Your Subscription**:

   * Pick the appropriate **Azure subscription** under which you want to create the function.

3. **Create or Select Resource Group**:

   * Click **“Create new”** to make a new resource group, or choose an **existing** one.

4. **Function App Name**:

   * Enter a **globally unique name**, e.g., `func109`.
     *(You can replace this with any name that hasn’t been taken globally.)*

5. **Region**:

   * Select **Central US**.
     *(This region allows trigger creation via Azure Portal.)*

6. **Runtime Stack**:

   * Choose **Node.js**.

7. **Version**:

   * Select **Node.js v20 (LTS)**.

8. **Instance Size**:

   * Set to **2048 MB (2 GB)**.

9. Click **“Next”** to proceed.

---

### ✅ **Step 3: Hosting Settings**

1. **Storage Account**:

   * Select an **existing storage account** or click **“Create new”**.

2. **Public Access**:

   * Set to **Enabled**.

3. **Virtual Network Integration**:

   * Set to **Disabled**.

---

### ✅ **Step 4: Monitoring**

1. **Application Insights**:

   * Set to **Enabled** for logging and diagnostics.

---

### ✅ **Step 5: Deployment Settings**

1. **Continuous Deployment**:

   * Set to **Disabled** (you can enable it later if needed).

---

### ✅ **Step 6: Review and Create**

1. Leave all other settings at their default values.
2. Click **“Review + Create”**.
3. After validation, click **“Create”**.

⏱️ The function app will take approximately **3–5 minutes** to be deployed.

---

Here's a clear and concise documentation of the steps you outlined for creating and testing an **HTTP-triggered Azure Function** using the **Azure Portal**:

---

# 📘 **Create and Test an HTTP-Triggered Azure Function in Azure Portal**

---

## ✅ **Step 1: Navigate to the Function App**

1. Open the [Azure Portal](https://portal.azure.com).
2. Go to your previously created **Function App** (e.g., `func109`).
3. In the left menu, click on **Functions**.
4. Click **+ Add** to create a new function.

---

## ✅ **Step 2: Create a Function**

1. Select the **"Develop in portal"** option.
2. Under **Template**, choose **Http trigger**.
3. Give your function a name (e.g., `HttpTrigger1` — you can choose any name).
4. Set **Authorization level** to **Function**.
5. Click **Create**.

---

## ✅ **Step 3: Default Function Code**

Once created, you’ll see a pre-generated function with this code:

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    const name = (req.query.name || (req.body && req.body.name));
    const responseMessage = name
        ? "Hello, " + name + ". This HTTP triggered function executed successfully."
        : "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.";

    context.res = {
        // status: 200, /* Defaults to 200 */
        body: responseMessage
    };
}
```

---

## ✅ **Step 4: Test the Function**

1. Click on **Test/Run** from the top bar.
2. In the **HTTP method**, select **POST**.
3. In the request body, paste:

```json
{
  "name": "Azure"
}
```

4. Click **Run**.

---

## ✅ **Step 5: Review the Response**

* **HTTP Status Code**: `200 OK`
* **Response Content**:

  ```
  Hello, Azure. This HTTP triggered function executed successfully.
  ```

---

## ✅ **Step 6: Get the Function URL**

1. Click on **Get function URL** (top-right).
2. Copy the URL — it will look like:

```
https://func109.azurewebsites.net/api/HttpTrigger1?code=YOUR_FUNCTION_KEY
```

3. Test it in your browser or a tool like Postman by appending `&name=raman`:

```
https://func109.azurewebsites.net/api/HttpTrigger1?code=...&name=raman
```

You should see:

```
Hello, raman. This HTTP triggered function executed successfully.
```

---

✅ Your HTTP-triggered Azure Function is now fully working!



---
