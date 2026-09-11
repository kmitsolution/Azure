# ACR Tasks — Azure Container Registry

**ACR Tasks** is one of the most useful features of Azure Container Registry because it allows you to **build and automate container images directly inside Azure**, without requiring Docker to be installed on your local machine or a dedicated build server.

Think of it as:

```text
Developer
   │
   │ Source Code
   ▼
ACR Task
   │
   │ docker build
   ▼
ACR
   │
   └── myapp:v1
```

Instead of:

```text
Developer Laptop
      │
      ├── Docker
      ├── Dockerfile
      │
      ▼
   docker build
      │
      ▼
 docker push
      │
      ▼
     ACR
```

ACR Tasks moves the **build process into Azure**.

---

# 1. Why do we need ACR Tasks?

Suppose you have:

```text
GitHub
   │
   │ source code
   ▼
Developer
   │
   │ docker build
   ▼
Docker Image
   │
   │ docker push
   ▼
ACR
```

The developer needs:

* Docker
* Dockerfile
* Docker build environment
* Docker credentials/authentication
* A machine with enough resources

With ACR Tasks:

```text
GitHub / Local Source
        │
        ▼
    ACR Task
        │
        │ Build
        ▼
       ACR
        │
        ▼
   myapp:v1
```

Azure provides the build environment.

---

# 2. ACR Tasks vs ACR

This distinction is important.

### ACR

Primarily:

```text
Store
Manage
Secure
Distribute
```

container images.

### ACR Tasks

Provides:

```text
Build
Automate
Test
Trigger
```

container images.

So:

```text
                ACR
                 │
       ┌─────────┴─────────┐
       │                   │
       ▼                   ▼
   Registry             ACR Tasks
       │                   │
 Store images          Build images
 Pull images           Automate builds
```

---

# 3. Basic ACR Task

Let's use your existing registry:

```text
ACR = myreg07
```

First check:

```bash
az acr show \
  --name myreg07 \
  --query loginServer \
  -o tsv
```

Expected:

```text
myreg07.azurecr.io
```

---

# 4. Create a Dockerfile

Create a simple application directory:

```bash
mkdir acr-task-demo
cd acr-task-demo
```

Create:

```text
Dockerfile
```

with:

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Create:

```text
index.html
```

Example:

```html
<html>
<head>
    <title>ACR Tasks Demo</title>
</head>
<body>
    <h1>Hello from ACR Tasks!</h1>
    <p>Image built directly in Azure.</p>
</body>
</html>
```

Your directory:

```text
acr-task-demo/
│
├── Dockerfile
└── index.html
```

---

# 5. Build using ACR Tasks

Now comes the interesting part.

Run:

```bash
az acr build \
  --registry myreg07 \
  --image acrtask-demo:v1 \
  .
```

That's it.

You don't need:

```bash
docker build
docker tag
docker push
```

ACR Tasks performs the build and pushes the resulting image into your registry.

---

# 6. What happens behind the scenes?

When you execute:

```bash
az acr build \
  --registry myreg07 \
  --image acrtask-demo:v1 \
  .
```

the flow is approximately:

```text
Cloud Shell
    │
    │ Docker build context
    ▼
ACR Task
    │
    │ Read Dockerfile
    ▼
Build Image
    │
    ▼
myreg07.azurecr.io
    │
    └── acrtask-demo:v1
```

The `.` means:

> Send the current directory as the build context.

So Azure receives:

```text
Dockerfile
index.html
```

and performs the build.

---

# 7. Verify the image

Run:

```bash
az acr repository list \
  --name myreg07 \
  -o table
```

You should see:

```text
Result
----------------
acrtask-demo
mynginx
```

Then:

```bash
az acr repository show-tags \
  --name myreg07 \
  --repository acrtask-demo \
  -o table
```

Expected:

```text
v1
```

So now:

```text
myreg07.azurecr.io/acrtask-demo:v1
```

exists in your ACR.

---

# 8. Deploy the ACR Task image to ACI

This makes a very nice end-to-end lab:

```text
Dockerfile
    │
    ▼
ACR Task
    │
    ▼
ACR
myreg07.azurecr.io/acrtask-demo:v1
    │
    ▼
ACI
    │
    ▼
Browser
```

Create ACI:

```bash
az container create \
  --resource-group central-in-rg \
  --name acrtask-demo \
  --location centralindia \
  --os-type Linux \
  --image myreg07.azurecr.io/acrtask-demo:v1 \
  --registry-login-server myreg07.azurecr.io \
  --registry-username "$ACR_USER" \
  --registry-password "$ACR_PASSWORD" \
  --dns-name-label raman-acrtask-demo \
  --ports 80 \
  --ip-address Public \
  --cpu 1 \
  --memory 1.5
```

Then:

```bash
az container show \
  --resource-group central-in-rg \
  --name acrtask-demo \
  --query ipAddress.fqdn \
  -o tsv
```

Open the returned URL.

You should see:

```text
Hello from ACR Tasks!

Image built directly in Azure.
```

---

# 9. ACR Task with GitHub

This is where ACR Tasks becomes much more powerful.

Instead of:

```text
Local machine
     │
     ▼
ACR Task
```

you can connect source code:

```text
GitHub
   │
   │ commit
   ▼
ACR Task
   │
   │ build
   ▼
ACR
   │
   ▼
Container
```

For example:

```text
Developer
    │
    │ git push
    ▼
GitHub
    │
    │ Trigger
    ▼
ACR Task
    │
    ├── docker build
    ├── docker test
    └── docker push
          │
          ▼
         ACR
```

This is effectively a lightweight CI workflow.

---

# 10. Automated Build Trigger

One of the major features is **automated builds**.

For example:

```text
Git push
    │
    ▼
ACR Task triggered
    │
    ▼
Build image
    │
    ▼
Run tests
    │
    ▼
Push image
```

You can trigger builds based on:

### Source code changes

```text
Git commit
    ↓
Build
```

### Base image updates

```text
nginx base image updated
          ↓
ACR Task
          ↓
Rebuild application image
```

### Manual trigger

```bash
az acr run
```

This is useful for CI/CD pipelines.

---

# 11. `az acr build` vs `az acr run`

These commands are worth understanding.

## `az acr build`

Used primarily for building a container image.

```bash
az acr build \
  --registry myreg07 \
  --image myapp:v1 \
  .
```

Think:

```text
Build → Push to ACR
```

---

## `az acr run`

Runs an ACR Task using a task definition/command.

For example:

```bash
az acr run \
  --registry myreg07 \
  --cmd "echo Hello ACR Tasks" \
  /dev/null
```

Think:

```text
ACR Task execution environment
```

It can be used for more sophisticated workflows.

---

# 12. Build Arguments

You can pass build arguments.

Dockerfile:

```dockerfile
FROM nginx:latest

ARG APP_VERSION

RUN echo "Version: ${APP_VERSION}" > /usr/share/nginx/html/version.txt
```

Build:

```bash
az acr build \
  --registry myreg07 \
  --image myapp:v2 \
  --build-arg APP_VERSION=2.0 \
  .
```

---

# 13. Multiple Tags

You can tag the image with multiple versions.

For example:

```text
myapp:v1
myapp:latest
```

A build could produce:

```text
myreg07.azurecr.io/myapp:v1
myreg07.azurecr.io/myapp:latest
```

For production, I'd recommend version-specific tags such as:

```text
myapp:1.0.0
myapp:1.0.1
myapp:1.0.2
```

rather than relying only on:

```text
latest
```

---

# 14. ACR Tasks and Multi-Stage Docker Builds

ACR Tasks works very well with multi-stage Dockerfiles.

Example:

```dockerfile
FROM node:22 AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build


FROM nginx:latest

COPY --from=build /app/dist /usr/share/nginx/html
```

Then:

```bash
az acr build \
  --registry myreg07 \
  --image frontend:v1 \
  .
```

Azure performs the entire build remotely.

---

# 15. ACR Tasks + Security

A production workflow could look like:

```text
                    GitHub
                       │
                       ▼
                  ACR Task
                       │
                ┌──────┴──────┐
                │             │
                ▼             ▼
             Build          Test
                │             │
                └──────┬──────┘
                       │
                       ▼
                    Scan
                       │
                       ▼
                      ACR
                       │
                       ▼
              AKS / ACI / App Service
```

This is much closer to an enterprise container workflow.

---

# 16. ACR Tasks vs GitHub Actions

This is an important interview question.

### ACR Tasks

```text
Source
  ↓
ACR Task
  ↓
ACR
```

It's tightly integrated with Azure Container Registry.

### GitHub Actions

```text
GitHub
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
ACR
```

GitHub Actions provides a broader CI/CD platform.

So:

| ACR Tasks               | GitHub Actions           |
| ----------------------- | ------------------------ |
| Azure-native            | GitHub-native            |
| Strong ACR integration  | Broad CI/CD ecosystem    |
| Container-focused       | General automation       |
| Build images in Azure   | Can build anywhere       |
| Great for ACR workflows | Great for complete CI/CD |

They can also be used **together**.

---

# 17. ACR Tasks + ACI — Your Complete Lab

Since you've just learned ACI, this is the perfect exercise:

```text
                    Developer
                        │
                        │ Dockerfile
                        ▼
                   ACR Task
                        │
                   Build Image
                        │
                        ▼
              ┌──────────────────┐
              │       ACR        │
              │                  │
              │ acrtask-demo:v1  │
              └────────┬─────────┘
                       │
                    AcrPull
                       │
                       ▼
              ┌──────────────────┐
              │       ACI        │
              │                  │
              │     NGINX        │
              └────────┬─────────┘
                       │
                       ▼
                    Browser
```

This gives students an end-to-end understanding:

**Source → Build → Registry → Container Runtime**

---

## Recommended next ACR Tasks labs

I'd teach them in this order:

```text
ACR Tasks
│
├── 1. What is ACR Tasks?
│
├── 2. az acr build
│
├── 3. Build image from Dockerfile
│
├── 4. Build arguments
│
├── 5. Multi-stage Docker build
│
├── 6. ACR Task + GitHub
│
├── 7. Automatic build on Git commit
│
├── 8. Base image update trigger
│
├── 9. ACR Task variables/secrets
│
├── 10. Build + Test workflow
│
├── 11. ACR Tasks + ACI
│
└── 12. ACR Tasks + AKS CI/CD
```

**The best next hands-on lab for you is #6: GitHub → ACR Task → ACR → ACI**, because it will turn what you've learned so far into a small CI/CD pipeline.
