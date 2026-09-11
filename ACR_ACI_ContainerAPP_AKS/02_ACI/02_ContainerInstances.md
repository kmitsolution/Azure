
# Lecture — Create Container from ACR using Azure Container Groups

## 1. Architecture

The overall flow is:

```text
                    Microsoft Entra ID
                           │
                           ▼
                  Managed Identity
                           │
                           ▼
┌──────────────────┐   Pull Image   ┌──────────────────────┐
│ Azure Container  │ ─────────────► │ Azure Container      │
│ Registry (ACR)   │                │ Instance (ACI)       │
│                  │                │                      │
│ myreg07          │                │ Container Group      │
│                  │                │   └── nginx          │
│ mynginx:latest   │                │                      │
└──────────────────┘                └──────────┬───────────┘
                                               │
                                               │ HTTP :80
                                               ▼
                                           Internet
```

### Important distinction

| Service              | Responsibility                                            |
| -------------------- | --------------------------------------------------------- |
| **ACR**              | Stores container images                                   |
| **ACI**              | Runs containers                                           |
| **Container Group**  | Logical group containing one or more containers           |
| **Managed Identity** | Allows ACI to authenticate to Azure resources such as ACR |

For this lab:

```text
ACR
Name: myreg07

Image:
myreg07.azurecr.io/mynginx:latest

ACI Container Group:
Name: mynginx-container-group
```

---

# Part 1 — Create Container Group from ACR using Azure Portal

## Step 1 — Open Container Instances

Go to:

**Azure Portal → Container Instances**

Click:

**+ Create**

Select:

**Azure Container Instance**

---

# Step 2 — Basics

Configure:

### Subscription

```text
Subscription 1
```

### Resource Group

Use your existing resource group:

```text
central-in-rg
```

### Container name

For example:

```text
mynginx-container-group
```

### Region

Choose:

```text
Central India
```

### Availability zones

For this simple lab, you can leave the default option.

---

# Step 3 — Image source

This is the important section.

Under **Image source**, select:

```text
Azure Container Registry
```

You should then be able to select your registry.

Select:

```text
myreg07
```

Then select:

```text
mynginx
```

And tag:

```text
latest
```

Your image should effectively be:

```text
myreg07.azurecr.io/mynginx:latest
```

---

# Step 4 — Container size

For an NGINX demonstration, you don't need much CPU or memory.

For example:

```text
CPU: 1
Memory: 1.5 GiB
```

The exact available values can vary by region and ACI configuration.

---

# Step 5 — Networking

Go to the networking section.

For a simple Internet-accessible demonstration:

### Network type

```text
Public
```

### DNS name label

Enter something unique, for example:

```text
raman-nginx-demo
```

This gives you a DNS name similar to:

```text
raman-nginx-demo.centralindia.azurecontainer.io
```

The exact hostname will depend on what Azure accepts and whether the name is available.

---

# Step 6 — Ports

NGINX listens on port:

```text
80
```

Add:

```text
Port: 80
Protocol: TCP
```

So the traffic flow becomes:

```text
Browser
   │
   │ HTTP :80
   ▼
ACI Public IP / DNS
   │
   ▼
NGINX Container :80
```

---

# Step 7 — ACR authentication

This is the most important part when the image is private.

Your ACR is:

```text
myreg07
```

and it contains:

```text
mynginx:latest
```

ACI needs permission to pull that image.

You have two main approaches.

### Option 1 — Registry username/password

ACI can authenticate using ACR credentials.

Conceptually:

```text
ACI
 │
 │ username/password
 ▼
ACR
 │
 ▼
mynginx:latest
```

This is easy for a lab, but **not the preferred production approach**.

### Option 2 — Managed Identity

Better architecture:

```text
ACI
 │
 │ Managed Identity
 ▼
Microsoft Entra ID
 │
 │ authorization
 ▼
ACR
 │
 │ AcrPull / repository permission
 ▼
mynginx
```

For your Azure learning lab, I recommend using **Managed Identity**, because it demonstrates the real-world authentication model.

---

# Step 8 — Assign Managed Identity

In the Container Instance creation flow, look for:

**Identity**

Enable:

```text
System assigned
```

or select an existing:

```text
User assigned managed identity
```

Since you've already been working with a UAMI, you can use:

```text
vm-acr-identity
```

if that identity is available in the portal.

However, remember:

> Creating/assigning the identity does NOT automatically give it permission to pull from ACR.

You must give the identity appropriate ACR permissions.

---

# Step 9 — Give identity ACR pull permission

Go to:

**Container Registry → myreg07 → Access control (IAM)**

Select:

**Add → Add role assignment**

For traditional ACR RBAC mode, select:

```text
AcrPull
```

Assign access to:

```text
Managed identity
```

Select your ACI managed identity.

For example:

```text
vm-acr-identity
```

Then:

**Review + assign**

---

# Step 10 — Review + Create

Click:

**Review + create**

Then:

**Create**

Azure now creates:

```text
Container Group
       │
       ▼
Container
       │
       ▼
mynginx:latest
       │
       ▼
myreg07.azurecr.io
```

---

# Part 2 — Verify from Portal

Open:

**Container Instances → mynginx-container-group**

You should see:

```text
State: Running
```

Click the container.

You should see information such as:

```text
Container name
Image
CPU
Memory
Port
IP address
FQDN
```

---

# Step 11 — Get the public IP

Inside the Container Group, look for:

```text
IP address
```

For example:

```text
20.x.x.x
```

Open:

```text
http://20.x.x.x
```

You should see the NGINX welcome page.

![Image](https://images.openai.com/static-rsc-4/6n8hPpyP2z3y0LZyWk4DBiIcKA1OgbfeXrcY8eEhCA6XB8HWPdEa8zVzPZXl14CXEBVo9JRlgWxqGhpFfNyqltv1y8bU5J2_6iFiWdegCEnxl7S7s_wd32z4FXG6uwoEqRJwqNCFs0L2kjWhakOf9WoumLyz-jDQZwnW6o8xcFFxO9B8D8pjio2LsfDdRh1x?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/qfkmhfgx_RfdIzBt3hAzl_YdHCyK4AMM76sTCA4yjA-ElABj8N3hOrYQaJUKtqYj5yQJlcAo4oLRCHYISl-iVmoHWV--6am0i0-YWjAnjKM7J4ytIEBQW8a_TkYXznErUcHHwtTmBOqLy9UzoCLxnpQ3IW1eli5WIooOzAfV7ssKaSr-3pNVq5IBfYJ05ku2?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/AdpaYsO9UXA3CfmdVlgqiz2dzYO6Eb1YTcd6vOvYYdf_bmXtYpNw21bMNShQlurc0FtlaO6_dVp9QOY94yEd7hpSM8lBQV8cPOwwcz8QIEO0Kb6Z_v0BJ3j3mIGN-Xi1QNbj28V_vMDcu1YOhiDAq1hnrX7xVChM4ge5w7e78dQmtbHQGy8xJd5LzJqPoWW7?purpose=fullsize)

---

# Part 3 — Create Container Group using Azure CLI

Now let's do exactly the same thing using CLI.

## Step 1 — Login

```bash
az login
```

Select your subscription:

```bash
az account set --subscription "Subscription 1"
```

Verify:

```bash
az account show -o table
```

---

# Step 2 — Define variables

```bash
RG="central-in-rg"
ACR_NAME="myreg07"
IMAGE="mynginx"
TAG="latest"
CONTAINER_GROUP="mynginx-container-group"
LOCATION="centralindia"
```

---

# Step 3 — Verify image exists

First check your repository:

```bash
az acr repository list \
  --name $ACR_NAME \
  -o table
```

You should see:

```text
mynginx
```

Check tags:

```bash
az acr repository show-tags \
  --name $ACR_NAME \
  --repository $IMAGE \
  -o table
```

You should see:

```text
latest
```

---

# Step 4 — Get ACR login server

```bash
az acr show \
  --name $ACR_NAME \
  --query loginServer \
  -o tsv
```

Expected:

```text
myreg07.azurecr.io
```

Therefore our image is:

```text
myreg07.azurecr.io/mynginx:latest
```

---

# Step 5 — Simple ACI deployment using ACR credentials

For your first CLI lab, you can use ACR credentials.

Get the username:

```bash
ACR_USER=$(az acr credential show \
  --name $ACR_NAME \
  --query username \
  -o tsv)
```

Get the password:

```bash
ACR_PASSWORD=$(az acr credential show \
  --name $ACR_NAME \
  --query "passwords[0].value" \
  -o tsv)
```

Then create the Container Group:

```bash
az container create \
  --resource-group $RG \
  --name $CONTAINER_GROUP \
  --location $LOCATION \
  --image myreg07.azurecr.io/mynginx:latest \
  --registry-login-server myreg07.azurecr.io \
  --registry-username "$ACR_USER" \
  --registry-password "$ACR_PASSWORD" \
  --dns-name-label raman-nginx-demo \
  --ports 80 \
  --ip-address Public \
  --cpu 1 \
  --memory 1.5
```

Azure will create:

```text
Container Group
        │
        └── mynginx
             │
             └── myreg07.azurecr.io/mynginx:latest
```

---

# Step 6 — Check Container Group

```bash
az container show \
  --resource-group $RG \
  --name $CONTAINER_GROUP \
  -o table
```

Check the state:

```bash
az container show \
  --resource-group $RG \
  --name $CONTAINER_GROUP \
  --query instanceView.state \
  -o tsv
```

Expected:

```text
Running
```

---

# Step 7 — Get IP address

```bash
az container show \
  --resource-group $RG \
  --name $CONTAINER_GROUP \
  --query ipAddress.ip \
  -o tsv
```

Example:

```text
20.204.x.x
```

Open:

```text
http://20.204.x.x
```

You should get:

**Welcome to nginx!**

---

# Step 8 — Get FQDN

```bash
az container show \
  --resource-group $RG \
  --name $CONTAINER_GROUP \
  --query ipAddress.fqdn \
  -o tsv
```

Example:

```text
raman-nginx-demo.centralindia.azurecontainer.io
```

Then:

```text
http://raman-nginx-demo.centralindia.azurecontainer.io
```

---

# Part 4 — Check Container Logs

This is extremely useful when teaching/troubleshooting ACI.

```bash
az container logs \
  --resource-group $RG \
  --name $CONTAINER_GROUP
```

You can also check the container events:

```bash
az container show \
  --resource-group $RG \
  --name $CONTAINER_GROUP \
  --query containers[0].instanceView.events
```

---

# Part 5 — Execute a command inside the container

You can execute commands inside a running ACI container.

For example:

```bash
az container exec \
  --resource-group $RG \
  --name $CONTAINER_GROUP \
  --exec-command "/bin/sh"
```

You'll get a shell inside the container.

Then:

```bash
nginx -v
```

or:

```bash
cat /etc/os-release
```

Exit:

```bash
exit
```

---

# Part 6 — Container Group Concept

This is important for your AZ-104 / Azure administration understanding.

A **Container Group** can contain multiple containers.

For example:

```text
             Container Group
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
     NGINX                  Application
     :80                       :8080
```

The containers in the same group share networking.

For example:

```text
Container Group IP
       │
       ├── nginx :80
       │
       └── app :8080
```

This is conceptually similar to the idea of a Kubernetes Pod, although ACI Container Groups and Kubernetes Pods are not identical.

---

# Part 7 — ACI + ACR + Managed Identity Architecture

This is the architecture I recommend you use in your advanced lab.

```text
                       Microsoft Entra ID
                              │
                              │
                    Managed Identity
                              │
                              ▼
                     ┌────────────────┐
                     │      ACR       │
                     │    myreg07     │
                     │                │
                     │ mynginx:latest │
                     └───────┬────────┘
                             │
                       Pull Image
                             │
                             ▼
                 ┌─────────────────────────┐
                 │ Azure Container Instance│
                 │                         │
                 │   Container Group       │
                 │        │                │
                 │        ▼                │
                 │      NGINX              │
                 │       :80               │
                 └──────────┬──────────────┘
                            │
                            ▼
                         Internet
```

The key point to explain to students is:

> **ACR stores the image. ACI runs the image. Managed Identity provides the identity used to authenticate to Azure resources. RBAC/ABAC determines whether that identity is allowed to pull the image.**

---

# Portal vs CLI Cheat Sheet

| Task        | Portal                       | CLI                                 |
| ----------- | ---------------------------- | ----------------------------------- |
| Create ACI  | Container Instances → Create | `az container create`               |
| Select ACR  | Image source → ACR           | `--image` + registry authentication |
| Set CPU     | Container size               | `--cpu`                             |
| Set memory  | Container size               | `--memory`                          |
| Expose port | Networking                   | `--ports 80`                        |
| Public IP   | Networking                   | `--ip-address Public`               |
| DNS         | DNS name label               | `--dns-name-label`                  |
| View status | Overview                     | `az container show`                 |
| View logs   | Containers → Logs            | `az container logs`                 |
| Shell       | Exec                         | `az container exec`                 |

---

# Important Production Note

For the classroom lab, using:

```bash
--registry-username
--registry-password
```

is easy to demonstrate.

But for a production architecture, prefer:

```text
Managed Identity
       ↓
Microsoft Entra ID
       ↓
ACR RBAC / ABAC
       ↓
Pull image
```

And don't put ACR passwords directly into scripts, source control, Dockerfiles, or CI/CD configuration.

---

## Your Hands-on Assignment

I recommend making this **ACI Assignment 1**:

### Task

1. Create an ACR named `myreg07`.
2. Push `mynginx:latest` into ACR.
3. Create an ACI Container Group named:

   ```text
   mynginx-container-group
   ```
4. Pull the image from:

   ```text
   myreg07.azurecr.io/mynginx:latest
   ```
5. Expose port `80`.
6. Configure a public IP.
7. Configure a DNS name.
8. Access NGINX from a browser.
9. View container logs.
10. Execute `/bin/sh` inside the container.
11. Delete the Container Group.
12. Recreate it using CLI.

Then **Assignment 2** can be much more interesting: **ACI + ACR + User Assigned Managed Identity + `AcrPull` + private networking**, which will connect directly with the Managed Identity and ACR concepts you've already covered.
