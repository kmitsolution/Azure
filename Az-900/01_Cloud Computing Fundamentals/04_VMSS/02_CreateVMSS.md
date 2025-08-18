Great — here's how you can **create an Azure Virtual Machine Scale Set (VMSS)** using different methods:

---

## ✅ 1. **Using Azure Portal (GUI method)**

### 🖥️ Steps:

1. Go to the [Azure Portal](https://portal.azure.com)
2. Search for **"Virtual machine scale sets"**
3. Click **“+ Create”**
4. Fill in the **Basics** tab:

   * Subscription & Resource Group
   * Scale set name
   * Region (e.g., East US)
   * Image (e.g., Ubuntu or Windows Server)
   * Admin username & password/SSH key
5. Choose the **Instance count** (e.g., start with 2 VMs)
6. Under **Scaling**, set up **automatic scaling rules** (optional):

   * Scale out when CPU > 70%
   * Scale in when CPU < 30%
7. Configure **Load Balancing** (optional but recommended)
8. Review and **create**

> 🟢 Azure will deploy the VMSS with identical VMs and manage scaling for you.

---

## ✅ 2. **Using Azure CLI**

Make sure you're logged in with:

```bash
az login
```

### 🖥️ Sample Command to Create a VMSS:

```bash
az vmss create \
  --resource-group MyResourceGroup \
  --name MyScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys
```

### Optional: Add autoscaling rule

```bash
az monitor autoscale create \
  --resource-group MyResourceGroup \
  --resource MyScaleSet \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscaleMyScaleSet \
  --min-count 2 \
  --max-count 5 \
  --count 2

az monitor autoscale rule create \
  --resource-group MyResourceGroup \
  --autoscale-name autoscaleMyScaleSet \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 1

az monitor autoscale rule create \
  --resource-group MyResourceGroup \
  --autoscale-name autoscaleMyScaleSet \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1
```

---

## ✅ 3. **Using ARM Template (Infrastructure as Code)**

You can define a VMSS in an Azure Resource Manager (ARM) template. Here’s a basic snippet:

```json
{
  "type": "Microsoft.Compute/virtualMachineScaleSets",
  "apiVersion": "2023-03-01",
  "name": "myScaleSet",
  "location": "[resourceGroup().location]",
  "sku": {
    "name": "Standard_DS1_v2",
    "capacity": 2,
    "tier": "Standard"
  },
  "properties": {
    "upgradePolicy": {
      "mode": "Automatic"
    },
    "virtualMachineProfile": {
      "storageProfile": {
        "imageReference": {
          "publisher": "Canonical",
          "offer": "UbuntuServer",
          "sku": "18.04-LTS",
          "version": "latest"
        }
      },
      "osProfile": {
        "computerNamePrefix": "vmss",
        "adminUsername": "azureuser",
        "linuxConfiguration": {
          "disablePasswordAuthentication": true,
          "ssh": {
            "publicKeys": [
              {
                "path": "/home/azureuser/.ssh/authorized_keys",
                "keyData": "<your-ssh-public-key>"
              }
            ]
          }
        }
      },
      "networkProfile": {
        // networking config here
      }
    }
  }
}
```

You'd deploy this using:

```bash
az deployment group create \
  --resource-group MyResourceGroup \
  --template-file vmss-template.json
```

---


