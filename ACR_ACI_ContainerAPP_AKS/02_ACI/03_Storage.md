For **ACI + Azure Files**, the most reliable approach today is to use **Azure CLI or an ARM/Bicep deployment**. The portal can still be used to create the Storage Account/File Share and inspect the resulting container, but the volume mount is much easier to demonstrate with CLI.

### Let's do the current CLI approach

You already have:

```text
Resource Group: central-in-rg
ACR:            myreg07
Image:          mynginx:latest
```

Create an Azure Storage Account:

```bash
az storage account create \
  --resource-group central-in-rg \
  --name acistorageraman01 \
  --location centralindia \
  --sku Standard_LRS \
  --kind StorageV2
```

Get the storage key:

```bash
STORAGE_KEY=$(az storage account keys list \
  --resource-group central-in-rg \
  --account-name acistorageraman01 \
  --query "[0].value" \
  -o tsv)
```

Create the file share:

```bash
az storage share-rm create \
  --resource-group central-in-rg \
  --storage-account acistorageraman01 \
  --name aci-share
```

Then create the ACI and mount the share:

```bash
az container create \
  --resource-group central-in-rg \
  --name mynginx-persistent \
  --location centralindia \
  --image myreg07.azurecr.io/mynginx:latest \
  --registry-login-server myreg07.azurecr.io \
  --registry-username "$ACR_USER" \
  --registry-password "$ACR_PASSWORD" \
  --dns-name-label raman-nginx-storage \
  --ports 80 \
  --ip-address Public \
  --cpu 1 \
  --memory 1.5 \
  --azure-file-volume-account-name acistorageraman01 \
  --azure-file-volume-account-key "$STORAGE_KEY" \
  --azure-file-volume-share-name aci-share \
  --azure-file-volume-mount-path /mnt/storage
```

The important parameters are:

```text
--azure-file-volume-account-name
--azure-file-volume-account-key
--azure-file-volume-share-name
--azure-file-volume-mount-path
```

After creation:

```bash
az container exec \
  --resource-group central-in-rg \
  --name mynginx-persistent \
  --exec-command "/bin/sh"
```

Then inside the container:

```bash
cd /mnt/storage
echo "Hello from persistent ACI storage" > test.txt
cat test.txt
```

Go to:

**Storage Account → `acistorageraman01` → Data storage → File shares → `aci-share`**

You should see:

```text
test.txt
```

Then we can delete the ACI, recreate it, mount the **same `aci-share`**, and prove that `test.txt` is still there.

### One important correction to the earlier explanation

The **Portal's current ACI creation wizard may not expose volume configuration at all**, so don't waste time looking for **Advanced → Volumes**. For this particular lab, use **CLI**.

If your goal is to teach this as an Azure course lecture, I would structure it as:

**Lecture: ACI Storage**

1. Container ephemeral filesystem
2. Why data is lost
3. Azure Storage Account
4. Azure File Share
5. Mount Azure Files into ACI
6. Write data
7. Delete ACI
8. Recreate ACI
9. Prove data survives
10. Compare **Azure Files vs Blob Storage**
11. Finally do the same deployment with **Managed Identity instead of Storage Account key**

That gives you a very strong real-world ACI storage lab.
