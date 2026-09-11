 **from scratch**, using both **Azure CLI** and the **Azure Portal**, and we'll use names that make the lab easy to understand.

We'll create:

```text
Resource Group:     rg-managed-identity
Managed Identity:   vm-acr-identity
VM:                 vm-1
```

The final architecture will be:

```text
                    Microsoft Entra ID
                           |
                           |
                 User Assigned Identity
                    vm-acr-identity
                           |
                           | Attached to
                           v
                         VM-1
                           |
                           | Token
                           v
                          ACR
```

---

# Part 1 — Create the User Assigned Managed Identity using CLI

## Step 1 — Login

Open **Azure Cloud Shell → Bash** or your local Azure CLI.

```bash
az login
```

Check your subscription:

```bash
az account show -o table
```

If you have multiple subscriptions:

```bash
az account list -o table
```

Set the required subscription:

```bash
az account set --subscription "Subscription 1"
```

Verify:

```bash
az account show --query "{Name:name,SubscriptionId:id,TenantId:tenantId}" -o table
```

---

# Step 2 — Set variables

This makes the remaining commands easier.

```bash
RG="central-in-rg"
IDENTITY_NAME="vm-acr-identity"
```

I'm using your existing resource group `central-in-rg`.

If you want a new resource group instead:

```bash
az group create \
  --name rg-managed-identity \
  --location centralindia
```

Then use:

```bash
RG="rg-managed-identity"
```

---

# Step 3 — Create the User Assigned Managed Identity

Run:

```bash
az identity create \
  --name "$IDENTITY_NAME" \
  --resource-group "$RG" \
  --location centralindia
```

You should get JSON containing:

```text
clientId
principalId
id
location
name
```

For example:

```text
name:        vm-acr-identity
location:    centralindia
clientId:    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
principalId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Important

There are two IDs you should understand:

```text
Client ID
    ↓
Used by applications to identify the managed identity

Principal ID
    ↓
The identity's service principal/object in Microsoft Entra ID
```

For Azure RBAC assignments, you'll commonly work with the **principal ID**.

---

# Step 4 — Verify the identity

Run:

```bash
az identity show \
  --name "$IDENTITY_NAME" \
  --resource-group "$RG" \
  -o table
```

You should see:

```text
Name              Location       PrincipalId
----------------  -------------  --------------------------------
vm-acr-identity   centralindia   xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx
```

You can also get the Resource ID:

```bash
IDENTITY_ID=$(az identity show \
  --name "$IDENTITY_NAME" \
  --resource-group "$RG" \
  --query id \
  -o tsv)
```

Check it:

```bash
echo $IDENTITY_ID
```

It should look like:

```text
/subscriptions/<subscription-id>/resourceGroups/central-in-rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/vm-acr-identity
```

---

# Part 2 — Attach the Identity to Your Existing VM

Your VM is:

```text
VM: vm-1
```

We need to know the VM's resource group.

Run:

```bash
az vm list \
  --query "[].{Name:name,ResourceGroup:resourceGroup,Location:location}" \
  -o table
```

You should get something like:

```text
Name    ResourceGroup     Location
------  ----------------  -----------
vm-1    central-in-rg     centralindia
```

Set:

```bash
VM_NAME="vm-1"
VM_RG="central-in-rg"
```

---

# Step 5 — Attach the User Assigned Identity

Run:

```bash
az vm identity assign \
  --resource-group "$VM_RG" \
  --name "$VM_NAME" \
  --identities "$IDENTITY_ID"
```

If successful, Azure returns the VM identity information.

---

# Step 6 — Verify

Run:

```bash
az vm identity show \
  --resource-group "$VM_RG" \
  --name "$VM_NAME" \
  -o json
```

You should see:

```text
"type": "UserAssigned"
```

and something similar to:

```text
"userAssignedIdentities": {
    "/subscriptions/.../resourceGroups/central-in-rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/vm-acr-identity": {}
}
```

🎉 Your identity is now attached to the VM.

---

# Part 3 — Do the Same Thing from Azure Portal

Now let's do it using the Portal.

Go to:

**Azure Portal → Managed Identities**

Click:

**Create**

---

## Step 1 — Basics

Select:

### Subscription

```text
Subscription 1
```

### Resource Group

Use:

```text
central-in-rg
```

### Region

```text
Central India
```

### Name

```text
vm-acr-identity
```

Then click:

**Review + create**

and:

**Create**

---

# Step 2 — Open the Managed Identity

Go to:

**Managed Identities → vm-acr-identity**

You should see:

```text
Type:
Microsoft.ManagedIdentity/userAssignedIdentities
```

This confirms it's a **User Assigned Managed Identity**.

---

# Step 3 — Attach it to the VM

Go to:

**Virtual Machines → vm-1**

Then:

**Security → Identity**

or:

**Settings → Identity**

depending on the current Portal layout.

Select:

### User assigned

Then click:

**+ Add**

---

# Step 4 — Select Subscription

Choose:

```text
Subscription 1
```

Then search:

```text
vm-acr-identity
```

You should now see:

```text
vm-acr-identity
```

Select it.

Click:

**Add**

Azure will attach the identity to the VM.

---

# Step 5 — Verify in Portal

Go back to:

**VM → Identity → User assigned**

You should see:

```text
Name
-----------------
vm-acr-identity
```

Your architecture is now:

```text
+---------------------------+
|     User Assigned MI      |
|                           |
|     vm-acr-identity       |
|                           |
|     Principal ID          |
|     Client ID             |
+-------------+-------------+
              |
              | Assigned to
              |
              v
+---------------------------+
|          VM-1             |
|                           |
|   Managed Identity:       |
|   vm-acr-identity         |
+---------------------------+
```

---

# Part 4 — Very Important: Identity ≠ Permission

At this point, your VM **has an identity**, but the identity doesn't automatically have access to Azure resources.

For example:

```text
VM
 |
 | Has identity
 v
vm-acr-identity
 |
 | ❌ No ACR permission yet
 v
ACR
```

We need to assign a role.

For your ACR lab, suppose the VM needs to **pull container images**.

Then:

```text
vm-acr-identity
       |
       | AcrPull
       v
      ACR
```

---

# Part 5 — Assign AcrPull Using CLI

First get your ACR resource ID:

```bash
ACR_NAME="ramanacr2026"
```

Then:

```bash
ACR_ID=$(az acr show \
  --name "$ACR_NAME" \
  --query id \
  -o tsv)
```

Get the managed identity's principal ID:

```bash
PRINCIPAL_ID=$(az identity show \
  --name "$IDENTITY_NAME" \
  --resource-group "$RG" \
  --query principalId \
  -o tsv)
```

Check it:

```bash
echo $PRINCIPAL_ID
```

Now assign `AcrPull`:

```bash
az role assignment create \
  --assignee-object-id "$PRINCIPAL_ID" \
  --assignee-principal-type ServicePrincipal \
  --role AcrPull \
  --scope "$ACR_ID"
```

Now your architecture is:

```text
                    ACR
                     ^
                     |
                  AcrPull
                     |
                     |
              vm-acr-identity
                     ^
                     |
                  attached
                     |
                     |
                    VM
```

---

# Part 6 — Assign AcrPull Using Portal

You can also do this completely through the Portal.

Go to:

**Container Registries → your ACR**

Then:

**Access control (IAM)**

Click:

**Add → Add role assignment**

Select:

```text
Role:
AcrPull
```

Click:

**Next**

For **Assign access to**, select the appropriate managed identity option.

Click:

**Select members**

Choose:

```text
User assigned managed identity
```

Then select:

```text
vm-acr-identity
```

Click:

**Review + assign**

Now:

```text
VM
 ↓
vm-acr-identity
 ↓
AcrPull
 ↓
ACR
```

---

# Part 7 — Test the Identity from the VM

Now comes the really interesting part.

SSH into your VM:

```bash
ssh <username>@<VM-IP>
```

From inside the VM, you can request a token from the Azure Instance Metadata Service.

Run:

```bash
curl -H Metadata:true \
  "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/" 
```

You should receive JSON containing an access token.

This proves:

```text
VM
 ↓
Managed Identity
 ↓
Microsoft Entra ID
 ↓
Access Token
```

For ACR specifically, the authentication flow is slightly different because the Docker/ACR client obtains an ACR-compatible token based on the managed identity's Entra authentication.

---

# Complete Lab Architecture

At the end of this exercise:

```text
                         Microsoft Entra ID
                                |
                                |
                         User Assigned MI
                         vm-acr-identity
                                |
                                | Attached
                                |
                                v
                         +-------------+
                         |    VM-1     |
                         +------+------+
                                |
                                | Identity
                                |
                                v
                         +-------------+
                         |    AcrPull  |
                         +------+------+
                                |
                                v
                         +-------------+
                         |     ACR     |
                         |             |
                         | frontend:v1 |
                         | backend:v1  |
                         +-------------+
```

## The 4 things students should remember

```text
1. Create User Assigned Managed Identity
                 ↓
2. Attach identity to VM
                 ↓
3. Give identity RBAC permission
                 ↓
4. VM uses identity to access Azure resource
```

And most importantly:

> **Managed Identity answers "Who am I?" RBAC answers "What am I allowed to do?"**

For example:

```text
vm-acr-identity
       ↓
       "I am this identity"
       ↓
       AcrPull
       ↓
       "I am allowed to pull images"
       ↓
       ACR
```


## Pull docker image on vm
Yes — this error is actually expected if the VM has the Managed Identity attached but **Docker has not authenticated to ACR using that identity**.

Your current situation is:

```text
VM1
 |
 | Managed Identity
 v
vm-acr-identity
 |
 | AcrPull
 v
myreg07.azurecr.io
```

But when you run:

```bash
docker pull myreg07.azurecr.io/mynginx
```

Docker doesn't automatically know how to use the VM's Managed Identity.

You need to authenticate Docker to ACR first.

## Step 1 — Verify the VM has the identity

On **VM1**, run:

```bash
az vm identity show \
  --name vm1 \
  --resource-group <VM_RESOURCE_GROUP>
```

You should see:

```text
"type": "UserAssigned"
```

and:

```text
"userAssignedIdentities": {
    ".../userAssignedIdentities/vm-acr-identity": {}
}
```

---

## Step 2 — Verify the identity has `AcrPull`

From Cloud Shell or Azure CLI, run:

```bash
az acr show \
  --name myreg07 \
  --query id \
  -o tsv
```

Then:

```bash
az role assignment list \
  --scope $(az acr show --name myreg07 --query id -o tsv) \
  --all \
  -o table
```

Look for your managed identity and:

```text
AcrPull
```

You want:

```text
vm-acr-identity
        |
        +---- AcrPull
                 |
                 v
               myreg07
```

---

# Step 3 — Login to ACR from VM1

This is the important step.

On **VM1**, run:

```bash
az login --identity
```

This tells Azure CLI:

> "Don't use my username/password. Authenticate this VM using its Managed Identity."

If you have **multiple managed identities** attached to the VM, specify the client ID:

```bash
az login --identity --client-id <CLIENT_ID>
```

If you only attached `vm-acr-identity`, simply use:

```bash
az login --identity
```

---

# Step 4 — Verify Azure CLI identity

Run:

```bash
az account show -o table
```

You should see the managed identity as the authenticated identity.

---

# Step 5 — Login Docker to ACR

Now run:

```bash
az acr login --name myreg07
```

You should get:

```text
Login Succeeded
```

This is the missing step in your current workflow.

The complete flow is:

```text
VM1
 |
 | Managed Identity
 v
Microsoft Entra ID
 |
 | Authentication
 v
az acr login
 |
 | Docker authentication
 v
myreg07.azurecr.io
```

---

# Step 6 — Pull the image

Now run:

```bash
docker pull myreg07.azurecr.io/mynginx:latest
```

You should see something like:

```text
latest: Pulling from mynginx
...
Status: Downloaded newer image for myreg07.azurecr.io/mynginx:latest
```

Then:

```bash
docker images
```

You should see:

```text
REPOSITORY                     TAG       IMAGE ID
myreg07.azurecr.io/mynginx    latest    xxxxx
```

---

# If `az login --identity` gives an error

Run this on VM1:

```bash
az login --identity
```

If it says something like:

```text
No identity assigned to this resource
```

then the Managed Identity isn't actually attached to VM1.

If it logs in successfully but:

```bash
az acr login --name myreg07
```

fails with authorization, then the problem is almost certainly the **ACR RBAC assignment**.

---

# One more important thing: your ACR permission mode

Because you've been learning:

**RBAC Registry + ABAC Repository Permissions**

there's an important distinction.

If your `myreg07` ACR is configured for:

```text
RBAC Registry + ABAC Repository Permissions
```

then **the old `AcrPull` role isn't necessarily the role you should use for repository-specific access**.

For the newer repository-level ABAC model, use the appropriate **Container Registry Repository Reader** role and specify the repository condition.

For example:

```text
VM Managed Identity
        |
        | Container Registry Repository Reader
        | Repository = mynginx
        v
myreg07
   |
   +---- mynginx:v1
```

Then:

```bash
docker pull myreg07.azurecr.io/mynginx:latest
```

is allowed.

But:

```bash
docker pull myreg07.azurecr.io/backend:latest
```

would be denied if the identity isn't authorized for `backend`.

---

## ⭐ Your troubleshooting sequence

Run these **in this exact order on VM1**:

```bash
az login --identity
```

Then:

```bash
az acr login --name myreg07
```

Then:

```bash
docker pull myreg07.azurecr.io/mynginx:latest
```

### If Step 1 succeeds but Step 2 fails

Send me the output of:

```bash
az acr login --name myreg07
```

### If Step 2 succeeds but Step 3 fails

Send me:

```bash
az acr show --name myreg07 --query "{name:name,loginServer:loginServer,roleAssignmentMode:roleAssignmentMode}" -o json
```


This is the exact pattern I'd recommend using for your **ACR + VM practical lab** before moving on to **ACR + AKS Managed Identity**.
