This is an important AZ-104 and Azure VM provisioning topic. These three methods are often confusing:

1. **Custom Data**
2. **Cloud-Init**
3. **Custom Script Extension**
4. **User Data** (newer feature)

Let's understand them clearly.

---

# Big Picture

```text
VM Creation Time
        │
        ├── Custom Data
        ├── Cloud-init
        ├── User Data
        │
VM Created
        │
        └── Custom Script Extension
```

---

# 1. Custom Data

Custom Data is simply a **Base64 encoded file** that Azure passes to the VM during creation.

Azure itself DOES NOT execute it.

Execution depends on the OS.

For Linux images like Ubuntu, cloud-init usually processes it.

---

## Characteristics

| Feature                  | Custom Data                   |
| ------------------------ | ----------------------------- |
| Available after creation | Yes                           |
| Automatically executed   | No (unless cloud-init exists) |
| Can modify later         | No                            |
| Max size                 | 64 KB                         |
| Lifetime                 | Stays inside VM               |
| Used at                  | VM creation                   |

---

## Flow

```text
Azure Portal
      ↓
Custom Data
      ↓
Base64 Encoded
      ↓
VM Metadata Service
      ↓
cloud-init reads it
```

---

## Ubuntu Example

```yaml
#cloud-config
package_update: true
packages:
 - nginx
```

Azure stores this as custom data.

Ubuntu cloud-init executes it.

---

# Where is it stored?

Ubuntu:

```bash
sudo cat /var/lib/waagent/CustomData
```

or

```bash
sudo cat /var/lib/cloud/instance/user-data.txt
```

---

# Can we modify later?

❌ No.

You cannot update Custom Data after VM creation.

To change it:

* Recreate VM
* Create new image

---

# 2. Cloud-Init

Cloud-init is a Linux package.

It is not Azure specific.

AWS, GCP, OpenStack also use it.

Cloud-init reads:

* Custom Data
* User Data
* Metadata

and performs configuration.

---

## Cloud-init Responsibilities

* Install packages
* Create users
* Configure hostname
* Configure disks
* Run commands

---

## Example Ubuntu Cloud-init

```yaml
#cloud-config

package_update: true

packages:
  - nginx
  - git

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
```

---

## Azure CLI

```bash
az vm create \
  --resource-group rg-demo \
  --name ubuntu-vm \
  --image Ubuntu2204 \
  --custom-data cloud-init.yaml
```

---

## Lifecycle

```text
VM Created
      ↓
cloud-init runs ONCE
      ↓
Configuration completed
      ↓
Does not execute again
```

---

## Re-run cloud-init?

Possible manually:

```bash
sudo cloud-init clean
sudo reboot
```

---

---

# 3. User Data (New Feature)

User Data was introduced recently.

This is similar to AWS EC2 UserData.

UserData can be queried from IMDS.

---

## Major Difference

Custom Data is immutable.

User Data can be modified.

---

| Feature                | User Data |
| ---------------------- | --------- |
| Available through IMDS | Yes       |
| Can modify later       | Yes       |
| Base64 required        | Yes       |
| Automatically executed | No        |
| Persistent             | Yes       |

---

## UserData Flow

```text
Azure
     ↓
IMDS Endpoint
     ↓
VM Reads Data
```

---

### Retrieve UserData

```bash
curl -H Metadata:true \
"http://169.254.169.254/metadata/instance/compute/userData?api-version=2021-01-01&format=text"
```

---

## Lifetime

```text
VM Exists
      ↓
UserData Exists
```

Even after reboot.

---

## Updating UserData

```bash
az vm update
```

or ARM/Bicep/Terraform.

This is impossible with Custom Data.

---

# Example UserData

```json
{
 "Environment":"Production",
 "Application":"WebServer"
}
```

Application inside VM can read it dynamically.

---

---

# Custom Data vs UserData

| Feature                     | Custom Data | User Data |
| --------------------------- | ----------- | --------- |
| Available after VM creation | Yes         | Yes       |
| Can update                  | No          | Yes       |
| IMDS accessible             | No          | Yes       |
| Used by cloud-init          | Yes         | Possible  |
| Mutable                     | No          | Yes       |

---

# 4. Custom Script Extension (CSE)

This is completely different.

It runs AFTER VM creation.

Azure VM Agent executes it.

---

## Architecture

```text
Azure
   ↓
VM Agent
   ↓
Custom Script Extension
   ↓
Run Script
```

---

## Purpose

* Install software
* Configure application
* Download files
* Join domain

---

## Lifetime

Runs:

* Once
* On demand
* Re-run if extension updated

---

## Can Modify?

✅ Yes

Can update extension anytime.

---

# Ubuntu Example

```bash
az vm extension set \
  --resource-group rg1 \
  --vm-name vm1 \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings \
'{
 "commandToExecute":"apt update && apt install nginx -y"
}'
```

---

# Flow

```text
VM Running
      ↓
Install Extension
      ↓
Azure VM Agent
      ↓
Execute Script
```

---

# Example using Script File

```json
{
 "fileUris":[
   "https://storage.blob.core.windows.net/scripts/install.sh"
 ],
 "commandToExecute":"bash install.sh"
}
```

---

# Custom Script Extension Lifecycle

```text
VM Created
      ↓
Install Extension
      ↓
Execute Script
      ↓
Can rerun later
```

---

# Complete Comparison

| Feature                 | Custom Data | Cloud-init   | User Data | Custom Script Extension |
| ----------------------- | ----------- | ------------ | --------- | ----------------------- |
| Runs During VM Creation | Yes         | Yes          | Optional  | No                      |
| Runs After VM Creation  | No          | No           | Possible  | Yes                     |
| Can Update Later        | No          | No           | Yes       | Yes                     |
| Needs VM Agent          | No          | No           | No        | Yes                     |
| Linux Support           | Yes         | Yes          | Yes       | Yes                     |
| Windows Support         | Limited     | No           | Yes       | Yes                     |
| Lifetime                | Permanent   | One Time     | Permanent | On Demand               |
| Main Purpose            | Bootstrap   | Provision VM | Metadata  | Configuration           |

---

# Ubuntu Example Comparison

## Cloud-init

```yaml
#cloud-config
packages:
 - nginx
```

---

## UserData

```json
{
  "Environment":"Dev"
}
```

---

## Custom Script Extension

```bash
apt update
apt install nginx -y
```

---

# Windows Example (Install IIS)

---

# PowerShell Script

```powershell
Install-WindowsFeature `
   -Name Web-Server `
   -IncludeManagementTools
```

---

# Custom Script Extension

```powershell
az vm extension set `
   --resource-group rg1 `
   --vm-name winvm `
   --publisher Microsoft.Compute `
   --name CustomScriptExtension `
   --settings '{
      "commandToExecute":
      "powershell Install-WindowsFeature Web-Server"
   }'
```

---

# UserData Example (Windows)

```powershell
<powershell>

Install-WindowsFeature `
    Web-Server `
    -IncludeManagementTools

New-Item `
   C:\Temp\userdata.txt `
   -ItemType File

</powershell>
```

---

# AZ-104 Exam Memory Trick

```text
Custom Data
=
Initial immutable bootstrap data

Cloud-init
=
Linux engine that processes custom data

User Data
=
Mutable metadata for VM

Custom Script Extension
=
Run scripts after VM deployment
```

---

# Easy Interview Answer

```text
VM Creation
     │
     ├── Custom Data
     ├── Cloud-init
     └── UserData
VM Running
     │
     └── Custom Script Extension
```

This is usually the architecture diagram expected in AZ-104 interviews and certification questions.

