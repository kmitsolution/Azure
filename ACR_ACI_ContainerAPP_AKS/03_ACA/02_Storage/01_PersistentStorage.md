Absolutely. Below is the **corrected complete documentation**, incorporating the important distinction we identified: **creating an Azure Files volume and mounting that volume into the container are two separate steps**.

# Azure Container Apps – Persistent Storage with Azure Files

## 1. Overview

Azure Container Apps (ACA) containers have an ephemeral local filesystem. If a container is restarted, replaced, or a new revision is deployed, files written only inside the container filesystem should not be treated as persistent application data.

For persistent file storage, Azure Container Apps can mount an **Azure Files file share** into a container.

In this lab, we will:

1. Create an Azure Storage Account
2. Create an Azure Files file share
3. Configure the Azure Files share in the Container Apps Environment
4. Create a volume in the Container App
5. Mount the volume inside the container at `/mnt/data`
6. Deploy a simple Flask application
7. Write a file from the application
8. Verify the file from Azure Storage
9. Restart/update the application
10. Verify that the file still exists
11. Perform the same configuration using Azure CLI and YAML

---

# 2. Architecture

The complete architecture is:

```text
                    Azure Storage Account
                         │
                         │
                         ▼
                    File Share
                     appdata
                         │
                         │
                         ▼
              Container Apps Environment
                      con-env
                         │
                  Environment Storage
                      appfiles
                         │
                         ▼
                 Container App Volume
                       end1
                         │
                         ▼
                 Container Mount
                     /mnt/data
                         │
                         ▼
                  Flask Application
                         │
                         ▼
                /mnt/data/message.txt
```

The important relationship is:

```text
Storage Account
      ↓
Azure File Share
      ↓
ACA Environment Storage
      ↓
Container App Volume
      ↓
Container Mount Path
      ↓
Application
```

---

# 3. SMB or NFS – Which One Should We Use?

Azure Container Apps provides options such as:

* SMB
* NFS

For this lab, use:

**SMB**

### SMB

SMB is appropriate for a simple Azure Files persistent-storage demonstration and is the straightforward option for an Azure Files share.

### NFS

NFS is useful for Linux workloads and scenarios requiring NFS-specific behavior or performance characteristics.

For our Flask persistent-storage lab:

```text
Storage Type = SMB
```

---

# 4. Lab Resources

We will use the following names.

| Resource                   | Name               |
| -------------------------- | ------------------ |
| Resource Group             | `MYRG-India`       |
| Location                   | `centralindia`     |
| Container Apps Environment | `con-env`          |
| Container App              | `myapp`            |
| Storage Account            | `conappstorage090` |
| File Share                 | `appdata`          |
| Environment Storage Name   | `appfiles`         |
| Container Volume           | `end1`             |
| Container Mount Path       | `/mnt/data`        |

You can replace these names with your own.

---

# 5. Create Resource Group

## Azure Portal

Go to:

**Azure Portal → Resource Groups → Create**

Enter:

```text
Resource group:
MYRG-India

Region:
Central India
```

Click:

**Review + create → Create**

---

# 6. Create Storage Account

Go to:

**Azure Portal → Storage accounts → Create**

Use:

```text
Resource group:
MYRG-India

Storage account name:
conappstorage090

Region:
Central India

Performance:
Standard

Redundancy:
Locally-redundant storage (LRS)
```

Click:

**Review → Create**

---

# 7. Create Azure Files Share

Open:

**Storage Account → conappstorage090**

Go to:

**Data storage → File shares**

Click:

**+ File share**

Enter:

```text
Name:
appdata
```

Create the file share.

You should now have:

```text
Storage Account
└── conappstorage090
    └── File Share
        └── appdata
```

Initially the share can be empty.

---

# 8. Create Container Apps Environment

If you already have `con-env`, you can skip this section.

Portal:

**Container Apps → Create**

Create the environment:

```text
Environment:
con-env
```

Use:

```text
Resource Group:
MYRG-India

Region:
Central India
```

---

# 9. Important Concept – Environment Storage vs Container Volume

This is the most important part of this lab.

There are two different configurations.

## Step 1 – Environment Storage

First, associate the Azure Files share with the Container Apps Environment.

For example:

```text
Environment:
con-env

Storage:
appfiles
```

The storage points to:

```text
Storage Account:
conappstorage090

File Share:
appdata
```

## Step 2 – Container Volume

Next, create a volume inside the Container App:

```text
Volume:
end1
```

and associate it with the environment storage:

```text
end1 → appfiles
```

## Step 3 – Container Mount

Finally, mount the volume into the container:

```text
end1 → /mnt/data
```

Therefore:

```text
appfiles
   ↓
end1
   ↓
/mnt/data
```

Creating `end1` alone is not enough.

If Azure shows:

```text
The following volumes have no associated mounts
```

it means the volume exists but hasn't been mounted into the container.

---

# 10. Configure Environment Storage – Azure Portal

Open:

**Container Apps Environment → con-env**

Go to:

**Settings → Volume mounts**

Click:

**Add**

Select:

```text
Storage type:
SMB
```

Enter the storage information.

Example:

```text
Name:
appfiles

Storage account name:
conappstorage090

Storage account key:
<storage-account-key>

File share:
appdata

Access mode:
ReadWrite
```

Click:

**Add / Save**

The exact wording of the Portal buttons can change as Azure updates the Portal UI.

The current Azure Container Apps storage documentation uses an environment-level storage configuration followed by a volume mount in the individual Container App.

---

# 11. Where Do We Get the Storage Account Key?

Open:

**Storage Account → conappstorage090**

Go to:

**Security + networking → Access keys**

You will see:

```text
Key 1
Key 2
```

Copy one of the keys.

For example:

```text
Storage account:
conappstorage090

Storage key:
<your-key>
```

Use the key when configuring the SMB storage connection.

---

# 12. Configure Container App Volume

Now open:

**Container Apps → myapp**

Go to:

**Application → Volumes**

You may see a volume such as:

```text
Volume name    Type
end1           Azure Files
```

This means the volume has been created.

However, you may also see a warning:

```text
The following volumes have no associated mounts
in this revision's containers: end1
```

This means:

```text
Volume created       ✅
Volume mounted       ❌
```

We now need to mount it.

---

# 13. Mount the Volume to the Container

Go to:

**Application → Containers**

Select your container.

Edit the container configuration.

Find:

**Volume mounts**

Click:

**Add volume mount**

Configure:

```text
Volume:
end1

Mount path:
/mnt/data
```

Leave the sub-path empty unless you have a specific requirement.

The final configuration is:

```text
Volume:
end1

Mount path:
/mnt/data
```

Save the configuration.

---

# 14. Why Are We Using `/mnt/data`?

Our Flask application will use:

```python
FILE = "/mnt/data/message.txt"
```

Therefore:

```text
Application
     │
     ▼
/mnt/data/message.txt
     │
     ▼
Container volume
     │
     ▼
Azure Files
     │
     ▼
appdata/message.txt
```

The application does not need to know the Azure Storage Account name.

It simply sees:

```text
/mnt/data
```

as a normal filesystem directory.

---

# 15. Create Flask Application

Create a directory:

```text
storage-demo
```

Inside it create:

```text
app.py
```

Use:

```python
from flask import Flask

app = Flask(__name__)

FILE = "/mnt/data/message.txt"


@app.route("/")
def home():

    try:
        with open(FILE, "r") as f:
            message = f.read()

    except FileNotFoundError:
        message = "No persistent file found."

    return f"""
    <h1>Azure Container Apps Persistent Storage Demo</h1>

    <p>Message stored in Azure Files:</p>

    <h2>{message}</h2>
    """


@app.route("/write")
def write():

    with open(FILE, "w") as f:
        f.write("Hello! This file is stored on Azure Files.")

    return "File written to persistent storage."


app.run(host="0.0.0.0", port=5000)
```

---

# 16. Create Dockerfile

Create:

```text
Dockerfile
```

Content:

```dockerfile
FROM python:3.12-slim

RUN pip install --no-cache-dir flask

WORKDIR /app

COPY app.py .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# 17. Build Docker Image

Build:

```bash
docker build -t storage-demo:1.0 .
```

Check:

```bash
docker images
```

You should see:

```text
storage-demo    1.0
```

---

# 18. Push Image to Azure Container Registry

If using ACR:

```bash
az acr login --name <ACR_NAME>
```

Tag:

```bash
docker tag storage-demo:1.0 <ACR_NAME>.azurecr.io/storage-demo:1.0
```

Push:

```bash
docker push <ACR_NAME>.azurecr.io/storage-demo:1.0
```

Example:

```bash
docker tag storage-demo:1.0 myacr.azurecr.io/storage-demo:1.0

docker push myacr.azurecr.io/storage-demo:1.0
```

---

# 19. Deploy Container App

If `myapp` already exists, update it.

The Container App should have:

```text
Image:
<ACR_NAME>.azurecr.io/storage-demo:1.0

Target port:
5000

Ingress:
External
```

For the demo, external ingress makes it easy to access the application from the browser.

---

# 20. Configure Volume Using YAML

Create:

```text
storage-demo.yaml
```

Example:

```yaml
properties:
  managedEnvironmentId: /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/MYRG-India/providers/Microsoft.App/managedEnvironments/con-env

  configuration:
    ingress:
      external: true
      targetPort: 5000
      transport: http

  template:
    containers:
      - name: storage-demo
        image: <ACR_NAME>.azurecr.io/storage-demo:1.0

        resources:
          cpu: 0.25
          memory: 0.5Gi

        volumeMounts:
          - volumeName: azure-files-volume
            mountPath: /mnt/data

    volumes:
      - name: azure-files-volume
        storageType: AzureFile
        storageName: appfiles
```

Important:

```text
storageName: appfiles
```

must match the environment storage name that you created earlier.

Also:

```text
volumeName: azure-files-volume
```

must match:

```text
volumeMounts:
  volumeName: azure-files-volume
```

---

# 21. Update Existing Container App Using YAML

PowerShell:

```powershell
az containerapp update `
  --name myapp `
  --resource-group MYRG-India `
  --yaml storage-demo.yaml
```

Bash/Linux:

```bash
az containerapp update \
  --name myapp \
  --resource-group MYRG-India \
  --yaml storage-demo.yaml
```

This creates a new revision containing the volume mount.

---

# 22. Configure Environment Storage Using CLI

The Azure CLI command for associating an Azure Files share with the Container Apps Environment is:

```bash
az containerapp env storage set
```

Microsoft's current Azure CLI documentation lists the Container Apps environment storage commands under `az containerapp env storage`.

First obtain your Storage Account key.

### Bash/Linux

```bash
STORAGE_KEY=$(az storage account keys list \
  --resource-group MYRG-India \
  --account-name conappstorage090 \
  --query "[0].value" \
  --output tsv)
```

### PowerShell

```powershell
$STORAGE_KEY = az storage account keys list `
  --resource-group MYRG-India `
  --account-name conappstorage090 `
  --query "[0].value" `
  --output tsv
```

---

# 23. Associate Azure Files with ACA Environment

### Bash/Linux

```bash
az containerapp env storage set \
  --name con-env \
  --resource-group MYRG-India \
  --storage-name appfiles \
  --storage-type AzureFile \
  --azure-file-account-name conappstorage090 \
  --azure-file-account-key "$STORAGE_KEY" \
  --azure-file-share-name appdata \
  --access-mode ReadWrite
```

### PowerShell

```powershell
az containerapp env storage set `
  --name con-env `
  --resource-group MYRG-India `
  --storage-name appfiles `
  --storage-type AzureFile `
  --azure-file-account-name conappstorage090 `
  --azure-file-account-key $STORAGE_KEY `
  --azure-file-share-name appdata `
  --access-mode ReadWrite
```

This creates:

```text
ACA Environment
con-env

     ↓

Environment Storage
appfiles

     ↓

Azure Files
conappstorage090/appdata
```

---

# 24. Verify Environment Storage

Run:

### PowerShell

```powershell
az containerapp env storage list `
  --name con-env `
  --resource-group MYRG-India `
  --output table
```

### Bash/Linux

```bash
az containerapp env storage list \
  --name con-env \
  --resource-group MYRG-India \
  --output table
```

You should see something similar to:

```text
Name       StorageType    AzureFileShareName    AccessMode
---------  -------------  --------------------  ----------
appfiles   AzureFile      appdata               ReadWrite
```

You can also inspect it:

### PowerShell

```powershell
az containerapp env storage show `
  --name con-env `
  --resource-group MYRG-India `
  --storage-name appfiles
```

---

# 25. Important: CLI Does Not Replace the App-Level Mount

This is an important concept.

The command:

```bash
az containerapp env storage set
```

associates the Azure Files share with the **Container Apps Environment**.

It does NOT by itself mount `/mnt/data` inside your container.

You still need the application volume configuration:

```yaml
volumes:
  - name: azure-files-volume
    storageType: AzureFile
    storageName: appfiles
```

and:

```yaml
volumeMounts:
  - volumeName: azure-files-volume
    mountPath: /mnt/data
```

Therefore:

```text
az containerapp env storage set
              │
              ▼
Environment Storage
              │
              ▼
        appfiles
              │
              ▼
          YAML
              │
       ┌──────┴──────┐
       ▼             ▼
    volumes      volumeMounts
       │             │
       └──────┬──────┘
              ▼
          /mnt/data
```

This separation is why simply creating a volume can result in the Azure Portal message:

```text
The following volumes have no associated mounts
```

---

# 26. Get Container App URL

PowerShell:

```powershell
$FQDN = az containerapp show `
  --name myapp `
  --resource-group MYRG-India `
  --query properties.configuration.ingress.fqdn `
  --output tsv

$FQDN
```

Bash/Linux:

```bash
FQDN=$(az containerapp show \
  --name myapp \
  --resource-group MYRG-India \
  --query properties.configuration.ingress.fqdn \
  --output tsv)

echo $FQDN
```

Open:

```text
https://<FQDN>
```

---

# 27. Initial Test

When you first open:

```text
https://<FQDN>
```

you may see:

```text
Azure Container Apps Persistent Storage Demo

Message stored in Azure Files:

No persistent file found.
```

This is expected.

The file has not been created yet.

---

# 28. Write Data to Azure Files

Open:

```text
https://<FQDN>/write
```

The application executes:

```python
with open(FILE, "w") as f:
    f.write("Hello! This file is stored on Azure Files.")
```

where:

```text
FILE = "/mnt/data/message.txt"
```

You should see:

```text
File written to persistent storage.
```

---

# 29. Read the File

Now open:

```text
https://<FQDN>/
```

You should see:

```text
Azure Container Apps Persistent Storage Demo

Message stored in Azure Files:

Hello! This file is stored on Azure Files.
```

---

# 30. Verify from Azure Storage Account

Go to:

**Storage Account → conappstorage090 → Data storage → File shares → appdata**

You should see:

```text
message.txt
```

Open the file.

The content should be:

```text
Hello! This file is stored on Azure Files.
```

This proves that the application is writing to Azure Files.

---

# 31. Verify from Inside the Container

Azure Container Apps allows you to open a shell in the running container.

Run:

### PowerShell

```powershell
az containerapp exec `
  --name myapp `
  --resource-group MYRG-India `
  --command /bin/sh
```

### Bash/Linux

```bash
az containerapp exec \
  --name myapp \
  --resource-group MYRG-India \
  --command /bin/sh
```

Inside the container:

```bash
ls
```

Check the mounted directory:

```bash
ls -l /mnt/data
```

You should see:

```text
message.txt
```

Read the file:

```bash
cat /mnt/data/message.txt
```

Expected:

```text
Hello! This file is stored on Azure Files.
```

You can also verify the mount:

```bash
mount
```

or:

```bash
df -h
```

---

# 32. Test Persistence

Now we want to prove that the data is persistent.

The important point is that the file is stored in Azure Files rather than only in the container's local filesystem.

You can create a new revision or restart the application.

After the new revision becomes active, access:

```text
https://<FQDN>/
```

The application should still display:

```text
Hello! This file is stored on Azure Files.
```

The reason is:

```text
Old container
     │
     X
     │
     └── Container removed/replaced

Azure Files
     │
     │
     ▼
message.txt remains
     │
     ▼
New container
     │
     ▼
/mnt/data/message.txt
```

---

# 33. Container Local Storage vs Persistent Storage

## Container Local Storage

Suppose your application writes:

```text
/app/message.txt
```

This is part of the container's local filesystem.

It should not be used as persistent application storage.

Conceptually:

```text
Container
└── /app
    └── message.txt
```

If the container is replaced, that local data should not be relied upon.

---

## Azure Files

When we mount Azure Files:

```text
/mnt/data
```

the directory points to the Azure Files share.

Therefore:

```text
/mnt/data/message.txt
```

is stored in:

```text
Azure Storage
└── File Share
    └── appdata
        └── message.txt
```

---

# 34. Complete Architecture

```text
                    Internet
                       │
                       ▼
               Azure Container App
                       │
                       │
                 Flask Application
                       │
                       ▼
                  /mnt/data
                       │
                       ▼
                Container Volume
                     end1
                       │
                       ▼
               Environment Storage
                    appfiles
                       │
                       ▼
                  Azure Files
                       │
                       ▼
                 File Share
                   appdata
                       │
                       ▼
                 message.txt
```

---

# 35. Portal Configuration Summary

The Portal configuration should look conceptually like this:

## Storage Account

```text
Name:
conappstorage090
```

## File Share

```text
Name:
appdata

Protocol:
SMB
```

## Container Apps Environment

```text
Name:
con-env
```

## Environment Volume Mount / Storage

```text
Name:
appfiles

Storage Account:
conappstorage090

File Share:
appdata

Access:
ReadWrite
```

## Container App

```text
Name:
myapp
```

## Container Volume

```text
Volume name:
end1

Type:
Azure Files
```

## Container Mount

```text
Volume:
end1

Mount path:
/mnt/data
```

---

# 36. CLI Configuration Summary

The environment storage command:

```powershell
az containerapp env storage set `
  --name con-env `
  --resource-group MYRG-India `
  --storage-name appfiles `
  --storage-type AzureFile `
  --azure-file-account-name conappstorage090 `
  --azure-file-account-key $STORAGE_KEY `
  --azure-file-share-name appdata `
  --access-mode ReadWrite
```

Then the application YAML contains:

```yaml
volumes:
  - name: azure-files-volume
    storageType: AzureFile
    storageName: appfiles
```

and:

```yaml
volumeMounts:
  - volumeName: azure-files-volume
    mountPath: /mnt/data
```

---

# 37. Troubleshooting

## Problem 1 – "No persistent file found"

This normally means the file hasn't been created yet.

Call:

```text
/write
```

then access:

```text
/
```

---

## Problem 2 – Volume shows "no associated mounts"

If Azure displays:

```text
The following volumes have no associated mounts
```

you have created the volume but haven't mounted it.

Check:

```text
Container App
→ Containers
→ Container
→ Volume mounts
```

Add:

```text
Volume:
end1

Mount path:
/mnt/data
```

---

## Problem 3 – `appfiles` does not appear

Verify environment storage:

```powershell
az containerapp env storage list `
  --name con-env `
  --resource-group MYRG-India `
  --output table
```

If `appfiles` isn't listed, configure it with:

```powershell
az containerapp env storage set `
  --name con-env `
  --resource-group MYRG-India `
  --storage-name appfiles `
  --storage-type AzureFile `
  --azure-file-account-name conappstorage090 `
  --azure-file-account-key $STORAGE_KEY `
  --azure-file-share-name appdata `
  --access-mode ReadWrite
```

---

## Problem 4 – Permission denied

Check:

```text
Access mode = ReadWrite
```

Also verify that the Storage Account key is correct.

---

## Problem 5 – Application cannot write to `/mnt/data`

Check inside the container:

```bash
ls -ld /mnt/data
```

Then:

```bash
touch /mnt/data/test.txt
```

If the command fails, verify the volume and mount configuration.

---

# 38. Key Commands

### Environment storage

```bash
az containerapp env storage set
```

### List environment storage

```bash
az containerapp env storage list
```

### Show environment storage

```bash
az containerapp env storage show
```

### Update Container App

```bash
az containerapp update --yaml storage-demo.yaml
```

### Enter Container

```bash
az containerapp exec
```

---

# 39. Important Notes

### Important Note 1

**Environment storage and container volume are not the same thing.**

Environment storage:

```text
appfiles
```

Container volume:

```text
end1
```

Container mount:

```text
/mnt/data
```

---

### Important Note 2

Creating a volume does not automatically mount it into a container.

You need:

```text
Volume
  ↓
Volume Mount
  ↓
/mnt/data
```

---

### Important Note 3

For this lab, use:

```text
SMB
```

rather than NFS.

---

### Important Note 4

The application should write to:

```text
/mnt/data
```

because that is the mounted persistent volume.

---

### Important Note 5

Do not assume that a container's normal filesystem is persistent.

For persistent file storage, use an external storage service such as Azure Files.

---

# 40. Final Lab Flow

The complete lab can be remembered as:

```text
STEP 1
Create Storage Account
        │
        ▼
STEP 2
Create Azure File Share
        │
        ▼
STEP 3
Create/Use ACA Environment
        │
        ▼
STEP 4
Associate Azure Files with Environment
        │
        ▼
STEP 5
Create Container App Volume
        │
        ▼
STEP 6
Mount Volume into Container
        │
        ▼
STEP 7
Mount Path = /mnt/data
        │
        ▼
STEP 8
Application writes message.txt
        │
        ▼
STEP 9
Azure Files stores message.txt
        │
        ▼
STEP 10
Restart/New Revision
        │
        ▼
STEP 11
File is still available
```

## Final Concept

The key concept to remember is:

```text
Azure Files = Persistent Storage

ACA Environment = Makes the storage available to Container Apps

Volume = Defines the storage inside the Container App

Volume Mount = Makes the volume available at a path inside the container

/mnt/data = Directory used by our application
```

So the final relationship is:

```text
Azure Files
   ↓
appdata
   ↓
appfiles
   ↓
end1
   ↓
/mnt/data
   ↓
message.txt
```

This is the complete Azure Container Apps **persistent storage using Azure Files (SMB)** implementation.
