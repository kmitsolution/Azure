Exactly — this is where the **ACI Container Group** concept becomes important.

If you currently have **two separate Azure Container Instances**, they are **not automatically a Container Group**.

For example, you may currently have:

```text
ACI 1
mynginx-container-group
    └── nginx

ACI 2
mynginx-persistent
    └── nginx
```

These are **two separate container groups**, each containing one container.

## What is a Container Group?

A Container Group is a single ACI deployment containing **one or more containers that share the same lifecycle and network**.

For example:

```text
             Container Group
          my-web-container-group
                    │
          ┌─────────┴─────────┐
          │                   │
          ▼                   ▼
      Container 1         Container 2
        nginx                app
        :80                 :8080
          │                   │
          └─────────┬─────────┘
                    │
              Shared network
                    │
              Public IP / DNS
```

Think of it roughly like:

```text
ACI Container Group ≈ Kubernetes Pod
```

Not exactly the same, but it's a useful mental model.

---

# Can you group your existing two ACIs?

**No — you cannot take two already-created ACI container groups and simply put them into one container group.**

The Container Group is defined **when you create the ACI deployment**.

So if you currently have:

```text
Container Group A
   └── nginx

Container Group B
   └── nginx
```

you need to create a **new Container Group** containing both containers.

Then you can delete the two old standalone groups.

---

# Let's create a real Container Group

Let's make something more meaningful than two NGINX containers.

We'll create:

```text
             ACI Container Group
              myapp-container-group
                       │
            ┌──────────┴──────────┐
            │                     │
            ▼                     ▼
         nginx                 app
        port 80              port 8080
```

Both containers will be inside **one ACI Container Group**.

---

# Important networking concept

Suppose the Container Group gets:

```text
Public IP
20.x.x.x
```

You don't get:

```text
nginx → public IP 1
app   → public IP 2
```

Instead:

```text
                 Public IP
                    │
                    ▼
          Container Group
              20.x.x.x
                    │
          ┌─────────┴─────────┐
          │                   │
       nginx :80           app :8080
```

The containers share the network namespace.

They can communicate using:

```text
localhost
```

For example, nginx could communicate with the application using:

```text
localhost:8080
```

---

# Create a Container Group with CLI

Let's use your existing image:

```text
myreg07.azurecr.io/mynginx:latest
```

and create a second container in the same group.

### Container 1

```text
nginx
port 80
```

### Container 2

For demonstration, we'll use another NGINX container:

```text
nginx2
port 8080
```

The architecture will be:

```text
              myapp-container-group
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
          nginx1               nginx2
           :80                  :8080
             │                   │
             └─────────┬─────────┘
                       │
                  Same network
```

---

# Step 1 — Create the Container Group

You can create the first container and second container together using `az container create`.

For example:

```bash
az container create \
  --resource-group central-in-rg \
  --name myapp-container-group \
  --location centralindia \
  --os-type Linux \
  --image myreg07.azurecr.io/mynginx:latest \
  --registry-login-server myreg07.azurecr.io \
  --registry-username "myreg07" \
  --registry-password "YOUR_ACR_PASSWORD" \
  --dns-name-label raman-multi-container \
  --ports 80 8080 \
  --ip-address Public \
  --cpu 2 \
  --memory 2
```

But this creates **one container**, not two.

To define multiple containers in the same group, the clean approach is a **YAML deployment**.

---

# Step 2 — Create YAML

Create:

```text
aci-container-group.yaml
```

Example:

```yaml
apiVersion: 2019-12-01
location: centralindia
name: myapp-container-group
properties:
  osType: Linux

  imageRegistryCredentials:
    - server: myreg07.azurecr.io
      username: myreg07
      password: YOUR_ACR_PASSWORD

  containers:

    - name: nginx1
      properties:
        image: myreg07.azurecr.io/mynginx:latest
        ports:
          - port: 80
        resources:
          requests:
            cpu: 1
            memoryInGB: 1

    - name: nginx2
      properties:
        image: myreg07.azurecr.io/mynginx:latest
        ports:
          - port: 8080
        resources:
          requests:
            cpu: 1
            memoryInGB: 1

  ipAddress:
    type: Public
    dnsNameLabel: raman-multi-container
    ports:
      - protocol: TCP
        port: 80
      - protocol: TCP
        port: 8080

type: Microsoft.ContainerInstance/containerGroups
```

Then deploy:

```bash
az container create \
  --resource-group central-in-rg \
  --file aci-container-group.yaml
```

Now Azure creates:

```text
┌──────────────────────────────────────────────┐
│           ACI Container Group                │
│                                              │
│        myapp-container-group                 │
│                                              │
│   ┌────────────────┐   ┌────────────────┐    │
│   │    nginx1      │   │    nginx2      │    │
│   │                │   │                │    │
│   │     :80        │   │     :8080      │    │
│   └────────────────┘   └────────────────┘    │
│                                              │
│             Shared Network                   │
└──────────────────────────────────────────────┘
```

---

# But there is one problem with your current images

You are using NGINX:

```text
myreg07.azurecr.io/mynginx:latest
```

NGINX normally listens on **port 80**.

If you expose the second container as:

```text
8080
```

that doesn't automatically make NGINX listen on 8080.

You would need to configure NGINX accordingly.

For learning the Container Group concept, that's okay, but I'd recommend a better lab.

---

# Better real-world example

Let's create:

```text
             ACI Container Group
                    │
       ┌────────────┴─────────────┐
       │                          │
       ▼                          ▼
     NGINX                    Application
      :80                       :8080
       │                          │
       └──────────┬───────────────┘
                  │
             localhost
```

The idea is:

```text
Internet
   │
   ▼
NGINX :80
   │
   │ localhost:8080
   ▼
Application :8080
```

This is a much better demonstration of **why you would put multiple containers into one ACI Container Group**.

---

# What happens to your Azure Files volume?

This is another important point because you just created:

```text
acistorageraman01
       │
       └── aci-share
```

You can mount the same Azure File Share into multiple containers in the group.

For example:

```text
                Azure Files
                 aci-share
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
       nginx1                 app
          │                     │
          └──── /mnt/storage ───┘
```

Both containers can access the shared storage.

This is one of the interesting reasons for using a multi-container group.

---

# Verify your Container Group

After creating it:

```bash
az container show \
  --resource-group central-in-rg \
  --name myapp-container-group \
  -o json
```

To see just the containers:

```bash
az container show \
  --resource-group central-in-rg \
  --name myapp-container-group \
  --query "containers[].{Name:name,Image:properties.image}" \
  -o table
```

You should get something like:

```text
Name     Image
-------  ----------------------------------
nginx1   myreg07.azurecr.io/mynginx:latest
nginx2   myreg07.azurecr.io/mynginx:latest
```

---

# Very important distinction

Your current setup:

```text
ACI Group 1
 └── Container 1

ACI Group 2
 └── Container 2
```

is **not** the same as:

```text
ACI Group
 ├── Container 1
 └── Container 2
```

In the second architecture, the containers have:

* Shared lifecycle
* Shared network
* Shared IP
* Shared DNS endpoint
* Ability to communicate via localhost
* Ability to share mounted volumes

---

## One more important point

If your goal is specifically to **take the two existing containers you already created and combine them**, don't delete them yet.

We can first inspect your two current containers and then I'll give you the **exact YAML that recreates those two containers as a single Container Group**, including your **ACR image and Azure Files `/mnt/storage` mount**.
