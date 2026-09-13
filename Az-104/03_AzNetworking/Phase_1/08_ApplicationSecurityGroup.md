
# Azure Application Security Group (ASG)

An **Application Security Group** is a logical way to group **network interfaces (NICs)** of Azure VMs based on their application role.

Instead of writing NSG rules using individual IP addresses, you can write rules using **ASG names**.

### Simple definition

> **ASG allows you to group VM network interfaces logically and use those groups as the source or destination in NSG rules.**

Think of it as:

```text
ASG = Logical group of VM NICs
NSG = Security rules
```

---

# Real-Time Example

Suppose you have a 3-tier application:

```text
                    Internet
                       |
                       ↓
                 Web Servers
                10.0.1.4
                10.0.1.5
                       |
                       ↓
              Application Servers
                10.0.2.4
                10.0.2.5
                       |
                       ↓
                Database Servers
                10.0.3.4
                10.0.3.5
```

Create three ASGs:

```text
ASG-Web
ASG-App
ASG-DB
```

Then your NSG rules can say:

```text
Internet
   ↓
ASG-Web
   Port 80/443

ASG-Web
   ↓
ASG-App
   Port 8080

ASG-App
   ↓
ASG-DB
   Port 1433
```

Notice that we don't need to specify:

```text
10.0.2.4
10.0.2.5
```

in the NSG rule.

---

# Why ASG is Useful

Imagine tomorrow you add another application server:

```text
AppVM03
10.0.2.6
```

You simply add its NIC to:

```text
ASG-App
```

The existing NSG rule automatically applies.

You don't need to modify the NSG rule.

That's the main benefit.

---

# ASG Architecture

```text
                    NSG
                     |
        +------------+-------------+
        |            |             |
        ↓            ↓             ↓
    ASG-Web      ASG-App       ASG-DB
        |            |             |
        ↓            ↓             ↓
      NIC1         NIC3          NIC5
      NIC2         NIC4          NIC6
       |             |             |
      VM1           VM3           VM5
      VM2           VM4           VM6
```

---

# Important Point

An ASG **doesn't provide security by itself**.

It is simply a grouping mechanism.

The **NSG provides the actual security rule**.

```text
ASG
 ↓
Grouping

NSG
 ↓
Security rule
```

---

# Creating ASGs Through Azure Portal

Go to:

**Azure Portal → Application security groups**

Click:

**+ Create**

---

## ASG-Web

Enter:

```text
Subscription: Your Subscription
Resource Group: az104-rg
Name: ASG-Web
Region: Central India
```

Click **Review + Create → Create**.

Create two more:

```text
ASG-App
ASG-DB
```

---

# Attach VM NIC to ASG

Suppose you have:

```text
web-vm01
```

Go to:

**VM → Networking → Network settings**

Select the NIC.

Under **Application security groups**, select:

```text
ASG-Web
```

Save.

Do the same for:

```text
web-vm02 → ASG-Web

app-vm01 → ASG-App
app-vm02 → ASG-App

db-vm01 → ASG-DB
db-vm02 → ASG-DB
```

---

# Create NSG Rule

Now create an NSG.

Example:

```text
nsg-app
```

Go to:

**NSG → Inbound security rules → Add**

Suppose application servers should accept traffic only from web servers.

Configure:

```text
Source:
Application security group

Source ASG:
ASG-Web

Source port:
*

Destination:
Application security group

Destination ASG:
ASG-App

Destination port:
8080

Protocol:
TCP

Action:
Allow

Priority:
100

Name:
Allow-Web-To-App
```

The rule means:

```text
ASG-Web
   |
   | TCP 8080
   ↓
ASG-App
```

---

# Another Rule: App → Database

Create:

```text
Source:
ASG-App

Destination:
ASG-DB

Destination port:
1433

Protocol:
TCP

Action:
Allow
```

Architecture:

```text
Internet
   |
   | 80/443
   ↓
ASG-Web
   |
   | 8080
   ↓
ASG-App
   |
   | 1433
   ↓
ASG-DB
```

This is much easier to maintain than using IP addresses.

---

# Azure CLI

Let's create the same environment using CLI.

## 1. Create Resource Group

```bash
az group create \
  --name az104-rg \
  --location centralindia
```

---

# 2. Create ASGs

### Web

```bash
az network asg create \
  --resource-group az104-rg \
  --name ASG-Web
```

### App

```bash
az network asg create \
  --resource-group az104-rg \
  --name ASG-App
```

### Database

```bash
az network asg create \
  --resource-group az104-rg \
  --name ASG-DB
```

---

# 3. Get VM NIC

For example:

```bash
az vm show \
  --resource-group az104-rg \
  --name web-vm01 \
  --query "networkProfile.networkInterfaces[0].id" \
  --output tsv
```

You get something like:

```text
/subscriptions/.../networkInterfaces/web-vm01-nic
```

---

# 4. Associate NIC with ASG

```bash
az network nic ip-config update \
  --resource-group az104-rg \
  --nic-name web-vm01-nic \
  --name ipconfig1 \
  --application-security-groups ASG-Web
```

Now:

```text
web-vm01 NIC
     ↓
ASG-Web
```

Do the same for additional VMs.

---

# 5. Create NSG

```bash
az network nsg create \
  --resource-group az104-rg \
  --name nsg-app
```

---

# 6. Create Web → App Rule

```bash
az network nsg rule create \
  --resource-group az104-rg \
  --nsg-name nsg-app \
  --name Allow-Web-To-App \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs ASG-Web \
  --destination-asgs ASG-App \
  --destination-port-ranges 8080
```

The important part is:

```text
--source-asgs ASG-Web
--destination-asgs ASG-App
```

---

# 7. Create App → DB Rule

```bash
az network nsg rule create \
  --resource-group az104-rg \
  --nsg-name nsg-db \
  --name Allow-App-To-DB \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs ASG-App \
  --destination-asgs ASG-DB \
  --destination-port-ranges 1433
```

---

# IP Address vs ASG

Without ASG:

```text
Source: 10.0.1.4
Destination: 10.0.2.4
Port: 8080
```

You need another rule for:

```text
10.0.1.5 → 10.0.2.5
```

This becomes difficult to maintain.

With ASG:

```text
Source: ASG-Web
Destination: ASG-App
Port: 8080
```

Any NIC belonging to ASG-Web can communicate with any NIC belonging to ASG-App according to that rule.

---

# Very Important AZ-104 Point

An ASG contains **NICs**, not VMs directly.

Think:

```text
VM
 ↓
NIC
 ↓
ASG
```

Not:

```text
VM
 ↓
ASG
```

Technically, you associate the **NIC's IP configuration** with the ASG.

---

# ASG vs NSG

| ASG                                  | NSG                     |
| ------------------------------------ | ----------------------- |
| Logical grouping                     | Security filtering      |
| Groups NICs                          | Contains security rules |
| Doesn't allow/deny traffic by itself | Allows/denies traffic   |
| Used by NSG rules                    | Evaluates traffic       |
| Example: ASG-Web                     | Example: Allow TCP 443  |

### Easy memory trick

> **ASG = Who belongs to the application group?**

> **NSG = What traffic is allowed?**

---

# Real-World Example

Imagine a company has:

```text
100 Web Servers
50 App Servers
20 DB Servers
```

You don't want to create hundreds of IP-based NSG rules.

Instead:

```text
ASG-Web
   ↓
100 Web NICs

ASG-App
   ↓
50 App NICs

ASG-DB
   ↓
20 DB NICs
```

NSG:

```text
Internet → ASG-Web → TCP 443

ASG-Web → ASG-App → TCP 8080

ASG-App → ASG-DB → TCP 1433
```

When a new VM is deployed:

```text
New App VM
    ↓
Add NIC to ASG-App
```

The existing NSG rules automatically apply.

**That's the real power of Application Security Groups.**
