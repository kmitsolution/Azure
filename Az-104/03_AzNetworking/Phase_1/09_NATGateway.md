First, let me clarify one important Azure concept because it differs from AWS.

> **In Azure, there is no "Private Subnet" checkbox.**
> A subnet is considered **private** when the VMs in it **do not have Public IP addresses**.

The architecture we'll build is:

```text
                    Internet
                        │
                   NAT Gateway
                  (Public IP)
                        │
────────────────────────────────────────
Azure Virtual Network (10.0.0.0/16)

Private Subnet
10.0.1.0/24
        │
        └── VM (No Public IP)
             Private IP: 10.0.1.4
```

Result:

* ❌ Nobody from the Internet can RDP/SSH into the VM.
* ✅ The VM can access the Internet (Windows Update, apt update, download software, etc.) through the NAT Gateway.

This is one of the recommended Azure designs.

---

# Lab Objective

Create:

* Resource Group
* VNet
* Private Subnet
* VM without Public IP
* NAT Gateway
* Public IP for NAT Gateway
* Associate NAT Gateway with subnet
* Test Internet connectivity

---

# Step 1: Create Resource Group

Open Azure Portal.

Go to:

```
Resource Groups
```

Click:

```
Create
```

Example:

```
Resource Group

RG-NATDemo

Region

Central India
```

Click **Review + Create**.

---

# Step 2: Create VNet

Go to

```
Virtual Networks
```

Click

```
Create
```

### Basics

```
Resource Group

RG-NATDemo

Name

Demo-VNet

Region

Central India
```

---

### IP Addresses

Address Space

```
10.0.0.0/16
```

Create subnet

```
Private-Subnet

10.0.1.0/24
```

Finish the VNet creation.

---

# Step 3: Create VM

Go to

```
Virtual Machines

Create
```

---

### Basics

```
Name

PrivateVM

Image

Windows Server 2022

(or)

Ubuntu 24.04
```

---

### Administrator Account

Create username/password.

---

### Networking

Select

```
Demo-VNet

Private-Subnet
```

Now the important part:

For **Public inbound ports**, choose:

```
None
```

For **Public IP**, choose:

```
None
```

So the VM will have:

```
Private IP

10.0.1.4
```

Only.

Create the VM.

---

# Verify

Open VM.

Go to

```
Networking
```

You'll notice:

```
Private IP

10.0.1.4

Public IP

None
```

This is now effectively a **private VM**.

---

# Step 4: Try Internet

If it's a Windows VM:

Open **Run Command** from the Azure portal (or connect using Azure Bastion if you've deployed it).

Run:

```powershell
ping google.com
```

or

```powershell
curl https://www.microsoft.com
```

For Linux:

```bash
ping google.com

curl https://google.com
```

> Depending on Azure's current outbound access behavior for your VM deployment, Internet access may or may not work at this point. In production, you should **not rely on the default outbound access**. The recommended approach is to use a NAT Gateway.

---

# Step 5: Create Public IP

Go to

```
Public IP Addresses
```

Click

```
Create
```

Example:

```
Name

NAT-Public-IP

SKU

Standard

Assignment

Static
```

Create it.

---

# Step 6: Create NAT Gateway

Go to

```
NAT Gateway
```

Click

```
Create
```

Example:

```
Name

Demo-NAT

Region

Central India
```

Click **Next**.

---

### Public IP

Choose

```
NAT-Public-IP
```

Click **Next**.

---

### Subnet

Associate it with

```
Demo-VNet

Private-Subnet
```

Finish the creation.

---

# Architecture Now

```
Internet
     │
NAT Gateway
     │
Private Subnet
     │
Private VM
```

---

# Step 7: Test

Go to

```
Run Command
```

Windows

```powershell
Invoke-WebRequest https://www.microsoft.com
```

Linux

```bash
curl https://www.google.com
```

It should work.

---

# What About Inbound?

Try to RDP.

You cannot.

There is **no Public IP**.

```
Internet

↓

Private VM

❌ Not Possible
```

Even though the VM has Internet access, no one can initiate a connection from the Internet.

---

# How NAT Works

Suppose your VM has:

```
Private IP

10.0.1.4
```

NAT Gateway has:

```
20.50.100.10
```

When the VM accesses Google:

```
10.0.1.4

↓

NAT Gateway

↓

20.50.100.10

↓

Google
```

Google replies to:

```
20.50.100.10
```

The NAT Gateway translates the response back to:

```
10.0.1.4
```

This is **Source NAT (SNAT)**.

---

# Important Point

NAT Gateway supports **outbound connections only**.

```
VM

↓

Internet

✔ Allowed
```

```
Internet

↓

VM

✘ Blocked
```

It **does not** provide inbound access.

---

# Enterprise Scenario

```
Internet
     │
Azure Firewall
     │
Application Gateway
     │
Web Subnet

App Subnet
   │
NAT Gateway

DB Subnet
```

In many production environments:

* Web servers receive inbound traffic through an Application Gateway or Load Balancer.
* Application servers have no Public IPs and use a NAT Gateway for outbound access.
* Database servers usually have no Internet access at all unless specifically required.

---

# What You'll Learn from This Lab

After completing this lab, you'll understand:

* How to build a **private subnet** in Azure (by using VMs without Public IPs).
* How to provide secure outbound Internet access with a **NAT Gateway**.
* Why a NAT Gateway allows **outbound only** and blocks unsolicited inbound traffic.
* Why this pattern is recommended for enterprise application servers.

This is a classic scenario that is frequently covered in **AZ-104**, **AZ-700**, and Azure Solution Architect interviews.
