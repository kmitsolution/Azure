# Demo: Two Azure Container Apps in One Environment — Frontend and Backend

## 1. What We Are Going to Build

In this demo, we will create two small containerized applications:

* **Frontend Container App**
* **Backend Container App**

Both applications will run inside the same **Azure Container Apps Environment**.

The architecture will be:

```text
                         Internet
                            │
                            │ HTTPS
                            ↓
                    ┌───────────────┐
                    │ frontend-app  │
                    │ External      │
                    │ Ingress       │
                    └───────┬───────┘
                            │
                            │ HTTP
                            ↓
                    ┌───────────────┐
                    │ backend-app   │
                    │ Internal      │
                    │ Ingress       │
                    └───────────────┘

                    Both inside:
                       con-env
```

We will also create an Azure Container Registry:

```text
Resource Group
│
├── Container Registry
│    ├── frontend:1.0
│    └── backend:1.0
│
├── Container Apps Environment
│    └── con-env
│
├── Container App
│    └── frontend-app
│
└── Container App
     └── backend-app
```

---

# 2. Application Communication

The user will access only the frontend:

```text
Browser
   │
   ↓
frontend-app
   │
   │ HTTP request
   ↓
backend-app
   │
   ↓
Response
```

The backend will return:

```text
Hello from Azure Container Apps Backend!
```

The frontend will display that response.

> **Important Note:** The frontend will use **external ingress**, while the backend will use **internal ingress**. Therefore, users can access the frontend from the Internet, but the backend does not need to be publicly exposed.

---

# 3. Create the Backend Application

We will create a very small Python Flask application.

Directory structure:

```text
backend/
├── Dockerfile
└── app.py
```

---

## 4. Backend `app.py`

Create:

```text
backend/app.py
```

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Azure Container Apps Backend!"

app.run(host="0.0.0.0", port=5000)
```

The backend simply listens on port `5000` and returns a message.

---

# 5. Backend Dockerfile

Create:

```text
backend/Dockerfile
```

```dockerfile
FROM python:3.12-slim

RUN pip install --no-cache-dir flask

WORKDIR /app

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

The Dockerfile performs the following:

```text
Python base image
       ↓
Install Flask
       ↓
Create /app
       ↓
Copy app.py
       ↓
Expose port 5000
       ↓
Start Flask
```

---

# 6. Build the Backend Docker Image

From the `backend` directory:

### Bash / Linux

```bash
docker build -t backend:1.0 .
```

### PowerShell

```powershell
docker build -t backend:1.0 .
```

Both commands are the same because Docker CLI syntax works the same way here.

---

# 7. Test the Backend Locally

Run the container:

### Bash / Linux

```bash
docker run -p 5000:5000 backend:1.0
```

### PowerShell

```powershell
docker run -p 5000:5000 backend:1.0
```

Test it:

### Bash / Linux

```bash
curl http://localhost:5000
```

### PowerShell

```powershell
curl http://localhost:5000
```

Expected response:

```text
Hello from Azure Container Apps Backend!
```

---

# 8. Create the Frontend Application

Now we create a small frontend application that calls the backend.

Directory structure:

```text
frontend/
├── Dockerfile
└── app.py
```

---

# 9. Frontend `app.py`

Create:

```text
frontend/app.py
```

```python
from flask import Flask
import os
import requests

app = Flask(__name__)

BACKEND_URL = os.getenv("BACKEND_URL")

@app.route("/")
def home():
    response = requests.get(BACKEND_URL)

    return f"""
    <h1>Frontend Container App</h1>
    <p>Response from Backend:</p>
    <h2>{response.text}</h2>
    """

app.run(host="0.0.0.0", port=5000)
```

The important part is:

```python
BACKEND_URL = os.getenv("BACKEND_URL")
```

We don't hard-code the backend address.

The backend URL will be provided as an environment variable when we create the frontend Container App.

---

# 10. Frontend Dockerfile

Create:

```text
frontend/Dockerfile
```

```dockerfile
FROM python:3.12-slim

RUN pip install --no-cache-dir flask requests

WORKDIR /app

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# 11. Build the Frontend Docker Image

### Bash / Linux

```bash
docker build -t frontend:1.0 .
```

### PowerShell

```powershell
docker build -t frontend:1.0 .
```

---

# 12. Test Frontend Locally

For local testing, first start the backend:

```bash
docker run -p 5000:5000 backend:1.0
```

Then run the frontend in another terminal.

### Bash / Linux

```bash
docker run -p 5001:5000 \
  -e BACKEND_URL=http://host.docker.internal:5000 \
  frontend:1.0
```

### PowerShell

```powershell
docker run -p 5001:5000 `
  -e BACKEND_URL=http://host.docker.internal:5000 `
  frontend:1.0
```

Open:

```text
http://localhost:5001
```

Expected result:

```text
Frontend Container App

Response from Backend:

Hello from Azure Container Apps Backend!
```

---

# 13. Create Azure Container Registry

Now we will push both Docker images to Azure Container Registry.

Set the variables.

## Bash / Linux

```bash
RG="containerapp-rg"
LOCATION="centralindia"
ENV="con-env"
ACR="conacr$RANDOM"
```

## PowerShell

```powershell
$RG="containerapp-rg"
$LOCATION="centralindia"
$ENV="con-env"
$ACR="conacr$(Get-Random)"
```

> **Important Note:** ACR names must be globally unique in Azure. The random number helps create a unique registry name.

---

# 14. Create the Resource Group

## Bash / Linux

```bash
az group create \
  --name $RG \
  --location $LOCATION
```

## PowerShell

```powershell
az group create `
  --name $RG `
  --location $LOCATION
```

---

# 15. Create Azure Container Registry

We will use the **Basic** SKU for this simple demo.

## Bash / Linux

```bash
az acr create \
  --name $ACR \
  --resource-group $RG \
  --location $LOCATION \
  --sku Basic
```

## PowerShell

```powershell
az acr create `
  --name $ACR `
  --resource-group $RG `
  --location $LOCATION `
  --sku Basic
```

---

# 16. Get the ACR Login Server

## Bash / Linux

```bash
ACR_LOGIN_SERVER=$(az acr show \
  --name $ACR \
  --query loginServer \
  --output tsv)
```

## PowerShell

```powershell
$ACR_LOGIN_SERVER = az acr show `
  --name $ACR `
  --query loginServer `
  --output tsv
```

The result will look similar to:

```text
conacr12345.azurecr.io
```

---

# 17. Login to ACR

## Bash / Linux

```bash
az acr login --name $ACR
```

## PowerShell

```powershell
az acr login --name $ACR
```

---

# 18. Tag the Backend Image

We need to tag the local image with the ACR login server.

## Bash / Linux

```bash
docker tag backend:1.0 \
  $ACR_LOGIN_SERVER/backend:1.0
```

## PowerShell

```powershell
docker tag backend:1.0 `
  "$ACR_LOGIN_SERVER/backend:1.0"
```

---

# 19. Tag the Frontend Image

## Bash / Linux

```bash
docker tag frontend:1.0 \
  $ACR_LOGIN_SERVER/frontend:1.0
```

## PowerShell

```powershell
docker tag frontend:1.0 `
  "$ACR_LOGIN_SERVER/frontend:1.0"
```

---

# 20. Push the Backend Image

## Bash / Linux

```bash
docker push $ACR_LOGIN_SERVER/backend:1.0
```

## PowerShell

```powershell
docker push "$ACR_LOGIN_SERVER/backend:1.0"
```

---

# 21. Push the Frontend Image

## Bash / Linux

```bash
docker push $ACR_LOGIN_SERVER/frontend:1.0
```

## PowerShell

```powershell
docker push "$ACR_LOGIN_SERVER/frontend:1.0"
```

Now our ACR contains:

```text
ACR
│
├── backend:1.0
│
└── frontend:1.0
```

---

# 22. Create Container Apps Environment

We will create the environment named:

```text
con-env
```

## Bash / Linux

```bash
az containerapp env create \
  --name $ENV \
  --resource-group $RG \
  --location $LOCATION
```

## PowerShell

```powershell
az containerapp env create `
  --name $ENV `
  --resource-group $RG `
  --location $LOCATION
```

Now:

```text
containerapp-rg
│
├── ACR
│
└── con-env
```

---

# 23. Get ACR Credentials

For this introductory demo, we will use ACR username/password authentication.

Get the username.

## Bash / Linux

```bash
ACR_USERNAME=$(az acr credential show \
  --name $ACR \
  --query username \
  --output tsv)
```

## PowerShell

```powershell
$ACR_USERNAME = az acr credential show `
  --name $ACR `
  --query username `
  --output tsv
```

Get the password.

## Bash / Linux

```bash
ACR_PASSWORD=$(az acr credential show \
  --name $ACR \
  --query "passwords[0].value" \
  --output tsv)
```

## PowerShell

```powershell
$ACR_PASSWORD = az acr credential show `
  --name $ACR `
  --query "passwords[0].value" `
  --output tsv
```

> **Important Note:** Using ACR admin credentials keeps this introductory demo simple. In production environments, prefer **managed identity** with appropriate **AcrPull** permissions rather than storing registry credentials.

---

# 24. Create the Backend Container App

Now we'll create:

```text
backend-app
```

The backend will use **internal ingress**.

## Bash / Linux

```bash
az containerapp create \
  --name backend-app \
  --resource-group $RG \
  --environment $ENV \
  --image $ACR_LOGIN_SERVER/backend:1.0 \
  --registry-server $ACR_LOGIN_SERVER \
  --registry-username $ACR_USERNAME \
  --registry-password $ACR_PASSWORD \
  --target-port 5000 \
  --ingress internal
```

## PowerShell

```powershell
az containerapp create `
  --name backend-app `
  --resource-group $RG `
  --environment $ENV `
  --image "$ACR_LOGIN_SERVER/backend:1.0" `
  --registry-server $ACR_LOGIN_SERVER `
  --registry-username $ACR_USERNAME `
  --registry-password $ACR_PASSWORD `
  --target-port 5000 `
  --ingress internal
```

Notice:

```text
--ingress internal
```

The backend is not publicly exposed.

---

# 25. Get the Backend FQDN

## Bash / Linux

```bash
BACKEND_FQDN=$(az containerapp show \
  --name backend-app \
  --resource-group $RG \
  --query properties.configuration.ingress.fqdn \
  --output tsv)

echo $BACKEND_FQDN
```

## PowerShell

```powershell
$BACKEND_FQDN = az containerapp show `
  --name backend-app `
  --resource-group $RG `
  --query properties.configuration.ingress.fqdn `
  --output tsv

$BACKEND_FQDN
```

The result will be an Azure Container Apps internal hostname.

For example:

```text
backend-app.internal.xxxxx.azurecontainerapps.io
```

The exact value will be different for your environment.

---

# 26. Create the Frontend Container App

Now create:

```text
frontend-app
```

The frontend will use **external ingress**.

We will pass the backend URL using the `BACKEND_URL` environment variable.

## Bash / Linux

```bash
BACKEND_URL="https://$BACKEND_FQDN"

az containerapp create \
  --name frontend-app \
  --resource-group $RG \
  --environment $ENV \
  --image $ACR_LOGIN_SERVER/frontend:1.0 \
  --registry-server $ACR_LOGIN_SERVER \
  --registry-username $ACR_USERNAME \
  --registry-password $ACR_PASSWORD \
  --target-port 5000 \
  --ingress external \
  --env-vars BACKEND_URL=$BACKEND_URL
```

## PowerShell

```powershell
$BACKEND_URL = "https://$BACKEND_FQDN"

az containerapp create `
  --name frontend-app `
  --resource-group $RG `
  --environment $ENV `
  --image "$ACR_LOGIN_SERVER/frontend:1.0" `
  --registry-server $ACR_LOGIN_SERVER `
  --registry-username $ACR_USERNAME `
  --registry-password $ACR_PASSWORD `
  --target-port 5000 `
  --ingress external `
  --env-vars BACKEND_URL=$BACKEND_URL
```

---

# 27. Final Architecture

We now have:

```text
                         Internet
                            │
                            │ HTTPS
                            ↓
                    ┌───────────────┐
                    │ frontend-app  │
                    │ External      │
                    │ Ingress       │
                    └───────┬───────┘
                            │
                            │ HTTP
                            ↓
                    ┌───────────────┐
                    │ backend-app   │
                    │ Internal      │
                    │ Ingress       │
                    └───────────────┘

                    Both inside:
                       con-env
```

And the images are stored in:

```text
                  Azure Container Registry
                            │
                 ┌──────────┴──────────┐
                 │                     │
           frontend:1.0           backend:1.0
                 │                     │
                 ↓                     ↓
           frontend-app           backend-app
```

---

# 28. Get the Frontend URL

## Bash / Linux

```bash
FRONTEND_FQDN=$(az containerapp show \
  --name frontend-app \
  --resource-group $RG \
  --query properties.configuration.ingress.fqdn \
  --output tsv)

echo "https://$FRONTEND_FQDN"
```

## PowerShell

```powershell
$FRONTEND_FQDN = az containerapp show `
  --name frontend-app `
  --resource-group $RG `
  --query properties.configuration.ingress.fqdn `
  --output tsv

Write-Host "https://$FRONTEND_FQDN"
```

Open the displayed URL in your browser.

You should see:

```text
Frontend Container App

Response from Backend:

Hello from Azure Container Apps Backend!
```

---

# 29. What Happened Behind the Scenes?

The browser only communicates with the frontend:

```text
Browser
   │
   │ HTTPS
   ↓
frontend-app
```

The frontend application executes:

```python
response = requests.get(BACKEND_URL)
```

The request then goes to:

```text
frontend-app
      │
      │ Internal HTTP communication
      ↓
backend-app
```

The backend returns:

```text
Hello from Azure Container Apps Backend!
```

The frontend receives the response and displays it to the user.

---

# 30. Why Is the Backend Internal?

We don't want users to directly access:

```text
backend-app
```

Instead:

```text
                 Internet
                    │
                    ↓
              frontend-app
                    │
                    ↓
              backend-app
```

Only the frontend needs to be public.

This is a common microservices pattern.

For example, a production application could look like:

```text
                         Internet
                            │
                            ↓
                       Frontend
                            │
          ┌─────────────────┼─────────────────┐
          ↓                 ↓                 ↓
     Order API         Payment API       User API
          │                 │                 │
          └─────────────────┴─────────────────┘
```

The individual backend services don't necessarily need public endpoints.

---

# 31. Why Put Both Apps in the Same Environment?

Both applications are deployed into:

```text
con-env
```

So:

```text
Container Apps Environment
│
├── frontend-app
│
└── backend-app
```

The environment provides the common application and networking boundary.

You can have multiple Container Apps inside the same environment:

```text
con-env
│
├── frontend-app
├── backend-app
├── payment-app
├── order-app
└── notification-app
```

Each application can be deployed and scaled independently.

---

# 32. Important Concepts Demonstrated

| Concept                                  | Demo                          |
| ---------------------------------------- | ----------------------------- |
| Dockerfile                               | Yes                           |
| Docker image                             | Yes                           |
| Azure Container Registry                 | Yes                           |
| ACR image                                | `frontend:1.0`, `backend:1.0` |
| Container Apps Environment               | `con-env`                     |
| Multiple Container Apps                  | Yes                           |
| External ingress                         | `frontend-app`                |
| Internal ingress                         | `backend-app`                 |
| Application-to-application communication | Yes                           |
| Environment variable                     | `BACKEND_URL`                 |
| Private backend                          | Yes                           |
| Microservices architecture               | Yes                           |

---

# 33. Key Points to Remember

### Container Apps Environment

```text
con-env
```

The common environment in which our Container Apps run.

### Frontend Container App

```text
frontend-app
```

Public-facing application with:

```text
External Ingress
```

### Backend Container App

```text
backend-app
```

Internal application with:

```text
Internal Ingress
```

### ACR

Stores our Docker images:

```text
frontend:1.0
backend:1.0
```

### Communication

```text
Frontend
   ↓
Backend
```

using the backend's internal Container Apps address.

---

# 34. Complete Flow

```text
                    Dockerfiles
                    /         \
                   /           \
                  ↓             ↓
             Frontend        Backend
              Image           Image
                 │               │
                 └───────┬───────┘
                         ↓
                        ACR
                  ┌──────┴──────┐
                  ↓             ↓
             frontend:1.0   backend:1.0
                  │             │
                  ↓             ↓
            frontend-app   backend-app
                  │             │
                  └──────┬──────┘
                         ↓
                       con-env
                         │
                         ↓
                  Container Apps
```

At runtime:

```text
User
 │
 │ HTTPS
 ↓
Frontend Container App
 │
 │ HTTP
 ↓
Backend Container App
 │
 ↓
Response
 │
 ↓
Frontend
 │
 ↓
User
```

> **Important Note:** This demo introduces the basic **microservices pattern in Azure Container Apps**: one public frontend communicates with a private backend, while both applications run in the same Container Apps Environment. The next logical step is to replace the ACR username/password with **Managed Identity + AcrPull**, which is the more secure production approach.
