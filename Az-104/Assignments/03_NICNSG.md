Absolutely. **Assignment 3** can focus specifically on **Azure NIC + NSG**, with hands-on exercises around IP configuration, NIC management, NSG rules, Service Tags, My IP Address, Application Security Groups (ASGs), and the difference between **NIC-level and subnet-level NSGs**.

# Azure Networking – Assignment 3

## NIC, IP Address & NSG Practical Assignment

### 🎯 Objective

By completing this assignment, you should be able to:

* Understand Azure Network Interface (NIC)
* Understand Private and Public IP addresses
* Configure Dynamic and Static Private IP
* Attach and detach Public IP
* Understand NIC IP configurations
* Create and attach a secondary NIC to a VM
* Understand NSG
* Create inbound and outbound NSG rules
* Use **My IP Address** in NSG rules
* Use **Service Tags**
* Use **Application Security Groups (ASG)**
* Associate NSG with a NIC
* Associate NSG with a subnet
* Understand NSG evaluation
* Troubleshoot connectivity using NSG

---

# Part 1 – Understand the NIC

Start with this architecture:

```text
Azure VM
   │
   ▼
  NIC
   │
   ▼
Subnet
   │
   ▼
VNet
```

### Questions

1. What is an Azure Network Interface?
2. Why does a VM require a NIC?
3. Can a VM exist without a NIC?
4. Where is the private IP configured?
5. Where is the public IP associated?
6. Can one NIC have multiple IP configurations?
7. Can one VM have multiple NICs?
8. Why would a VM require multiple NICs?
9. Can you attach a NIC from one VNet to a VM in another VNet?
10. What happens to the NIC when a VM is deleted?

---

# Part 2 – Private IP Address

Create a VM and examine its NIC.

Find:

```text
NIC Name
Private IP
Private IP Allocation
Subnet
VNet
```

### Practical Task

Check whether the private IP is:

```text
Dynamic
```

or

```text
Static
```

### Task A – Change Dynamic → Static

Change the VM's private IP allocation from **Dynamic to Static**.

Record:

```text
Before:
Private IP = __________
Allocation = Dynamic

After:
Private IP = __________
Allocation = Static
```

### Questions

1. Does the IP address necessarily change when you change Dynamic to Static?
2. Why would an application require a static private IP?
3. Give three real-world use cases for static private IPs.

---

# Part 3 – Public IP Address

Now examine the VM's public IP.

```text
VM
 │
NIC
 │
IP Configuration
 │
Public IP
```

### Practical Task

1. Find the Public IP associated with your VM.
2. Record its name.
3. Record its IP address.
4. Check its allocation method.
5. Remove the Public IP association.
6. Verify whether you can still connect to the VM from the Internet.
7. Attach the Public IP again.
8. Verify connectivity.

### Questions

1. Is a public IP mandatory for every VM?
2. Can a VM communicate with another VM without a public IP?
3. Why would production VMs often not have public IP addresses?
4. What is the difference between public and private IP?
5. What is the security advantage of removing a public IP?

---

# Part 4 – Attach and Remove NIC

Now move to **multiple NICs**.

Create a second NIC:

```text
nic-vm01-secondary
```

Use the same:

```text
VNet
Subnet
```

as required by Azure's NIC/VM configuration rules.

### Practical Task

Attach the secondary NIC to your VM.

Your architecture should become:

```text
                 VM
              /     \
             /       \
         NIC-1       NIC-2
           │           │
           ▼           ▼
        Subnet       Subnet
```

Check:

```text
VM
 ├── NIC-1
 │    └── Private IP
 │
 └── NIC-2
      └── Private IP
```

### Questions

1. Why would an organization use multiple NICs?
2. Can every VM size support multiple NICs?
3. Can you attach an unlimited number of NICs?
4. What is the purpose of a secondary NIC?
5. Can NICs be attached to different subnets?
6. Can the NICs belong to different VNets?

### Important Task

Try to detach the NIC.

Determine:

> **Can the primary NIC of a VM be detached?**

Then determine how a **secondary NIC** can be detached.

---

# Part 5 – NIC IP Configuration

Open:

**VM → Networking → Network Interface → IP configurations**

Study the configuration.

You should identify:

```text
IP Configuration
       │
       ├── Private IP
       │
       └── Public IP
```

### Questions

1. What is `ipconfig1`?
2. What is a NIC IP configuration?
3. Can one NIC have multiple private IP configurations?
4. Can a secondary private IP have a public IP?
5. What is the difference between NIC and IP configuration?

---

# Part 6 – Introduction to NSG

Now create:

```text
NSG:
nsg-web
```

Understand:

```text
NSG
 │
 ├── Inbound Rules
 │
 └── Outbound Rules
```

### Questions

1. What is an NSG?
2. What is an inbound rule?
3. What is an outbound rule?
4. What is rule priority?
5. What happens when two rules have different priorities?
6. What happens when an Allow and Deny rule both apply?
7. What is the default priority range?
8. What are the default NSG rules?
9. Does an NSG state whether traffic is allowed or denied?
10. Does an NSG inspect application content like a full application firewall?

---

# Part 7 – NSG: My IP Address

This is an important practical exercise.

Create an inbound rule to allow RDP/SSH **only from your current public IP address**.

For example:

```text
Source:
My IP Address

Destination:
Any

Protocol:
TCP

Destination Port:
3389 or 22

Action:
Allow
```

### Practical Task

Create:

```text
Allow-Admin-Access
```

with your current public IP as the source.

Test connectivity.

Then change the source to:

```text
Any
```

Compare the behavior.

Finally change it back to:

```text
My IP Address
```

### Questions

1. What does **My IP Address** mean in the NSG rule creation screen?
2. Is it the VM's private IP?
3. Is it the VM's public IP?
4. Whose IP address is being specified as the source?
5. Why is restricting administrative access to your IP safer than allowing `Any`?

---

# Part 8 – NSG Service Tags

Now create an NSG rule using a **Service Tag**.

For example:

```text
Source:
Internet
```

or another appropriate Azure Service Tag depending on the scenario.

Research and explain these Service Tags:

```text
Internet
VirtualNetwork
AzureCloud
AzureLoadBalancer
Storage
Sql
```

### Questions

1. What is an Azure Service Tag?
2. Why are Service Tags useful?
3. Why is using a Service Tag better than maintaining a large list of IP addresses in some scenarios?
4. What does `VirtualNetwork` represent?
5. What does `AzureLoadBalancer` represent?
6. What is the difference between `Internet` and `VirtualNetwork`?

### Practical Scenario

You have a web server that should receive HTTP traffic from the Internet.

Design the NSG rule:

```text
Source: Internet
Destination: Web VM
Protocol: TCP
Port: 80
Action: Allow
```

---

# Part 9 – Application Security Group (ASG)

This is an important Azure networking concept.

Create two Application Security Groups:

```text
ASG-Web
ASG-App
```

Assign:

```text
Web VM
   ↓
ASG-Web

Application VM
   ↓
ASG-App
```

Architecture:

```text
                 VNet
                  │
        ┌─────────┴─────────┐
        │                   │
      Web VM              App VM
        │                   │
     ASG-Web             ASG-App
```

Now create an NSG rule:

```text
Source:
ASG-Web

Destination:
ASG-App

Protocol:
TCP

Port:
8080

Action:
Allow
```

### Questions

1. What is an Application Security Group?
2. Why use ASGs?
3. How is ASG different from NSG?
4. Does ASG itself allow or deny traffic?
5. Can ASG be used as a source in an NSG rule?
6. Can ASG be used as a destination in an NSG rule?
7. Why is this architecture easier to maintain than specifying individual VM IP addresses?

---

# Part 10 – NSG at NIC Level

Create:

```text
NSG-NIC
```

Associate it with:

```text
NIC of Web VM
```

Architecture:

```text
VNet
 │
Subnet
 │
 └── Web VM
       │
      NIC
       │
    NSG-NIC
```

Create:

```text
Allow TCP 80
```

and test HTTP connectivity.

Then create:

```text
Deny TCP 80
```

with a higher priority and observe the result.

### Questions

1. What does NIC-level NSG mean?
2. Which VM is affected?
3. Does the rule automatically affect every VM in the subnet?
4. What is the advantage of NIC-level NSG?
5. When would you use a NIC-level NSG?

---

# Part 11 – NSG at Subnet Level

Now create:

```text
NSG-Subnet
```

Associate it with:

```text
web-subnet
```

Architecture:

```text
VNet
 │
 └── Web Subnet
       │
       ├── VM1
       ├── VM2
       └── VM3
       
       NSG-Subnet
```

### Questions

1. Which VMs are affected?
2. Do you need to attach the NSG individually to every VM?
3. What is the advantage of subnet-level NSG?
4. When would you prefer subnet-level NSG?

---

# Part 12 – NIC NSG vs Subnet NSG 🔥

This is one of the most important parts of the assignment.

Create this architecture:

```text
                    VNet
                     │
                 Web Subnet
                     │
              NSG-Subnet
                     │
          ┌──────────┼──────────┐
          │          │          │
         VM1        VM2        VM3
          │          │          │
        NIC-1      NIC-2      NIC-3
          │          │          │
       NSG-NIC1
```

### Scenario

Subnet NSG:

```text
Allow TCP 80
```

NIC NSG:

```text
Deny TCP 80
```

### Question

Will HTTP traffic reach VM1?

**Explain why.**

Then reverse the configuration:

Subnet NSG:

```text
Deny TCP 80
```

NIC NSG:

```text
Allow TCP 80
```

### Question

Will HTTP traffic reach VM1?

**Explain your answer.**

This exercise should help you understand that **both NSG association points can affect traffic**.

---

# Part 13 – Real-World NSG Use Cases

For each scenario, design an NSG rule.

### Scenario 1 – Web Server

Allow:

```text
Internet → Web Server → TCP 80
```

---

### Scenario 2 – Secure Web

Allow:

```text
Internet → Web Server → TCP 443
```

Block:

```text
Internet → Web Server → TCP 22
```

---

### Scenario 3 – Administrator

Allow:

```text
My Public IP → VM → TCP 3389
```

---

### Scenario 4 – Application Tier

Allow:

```text
Web ASG → App ASG → TCP 8080
```

---

### Scenario 5 – Database

Allow:

```text
App ASG → Database → TCP 1433
```

Do **not** allow:

```text
Internet → Database → TCP 1433
```

---

### Scenario 6 – Azure Load Balancer

Research the appropriate Azure Service Tag and design an NSG rule for traffic originating from an Azure Load Balancer.

---

# Part 14 – Troubleshooting Challenge 🔥

You have:

```text
Web VM
Private IP: 10.0.1.4

App VM
Private IP: 10.0.2.4
```

The application team says:

> "Web VM cannot connect to App VM on TCP 8080."

You need to troubleshoot.

Check:

```text
1. VM is running
2. NIC exists
3. Correct VNet
4. Correct subnet
5. Private IP
6. NSG on NIC
7. NSG on subnet
8. NSG inbound rule
9. NSG outbound rule
10. Application is listening on port 8080
```

### Your Task

Create an NSG rule that allows:

```text
Source: Web subnet / ASG-Web
Destination: App subnet / ASG-App
Protocol: TCP
Port: 8080
Action: Allow
```

Then test connectivity.

---

# Part 15 – Final Practical Architecture

Build this environment:

```text
                         Internet
                            │
                            │ TCP 80/443
                            ▼
                    ┌──────────────┐
                    │  Web Subnet  │
                    │  NSG-Web     │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │    Web VM    │
                    │    NIC       │
                    │   ASG-Web    │
                    └──────┬───────┘
                           │
                        TCP 8080
                           │
                    ┌──────▼───────┐
                    │   App Subnet │
                    │   NSG-App    │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │    App VM    │
                    │    NIC       │
                    │   ASG-App    │
                    └──────┬───────┘
                           │
                        TCP 1433
                           │
                    ┌──────▼───────┐
                    │  DB Subnet   │
                    │   NSG-DB     │
                    └──────────────┘
```

Your NSG design should ensure:

```text
Internet → Web              ALLOW 80/443
Internet → App              DENY
Internet → Database         DENY

Web → App                   ALLOW 8080
Web → Database              DENY

App → Database              ALLOW 1433

Admin IP → Web              ALLOW management port
Admin IP → App              ALLOW management port
```

---

# 🧪 Assignment 3 – Test Yourself

Try answering these without looking at your notes.

### Basic

1. What is a NIC?
2. Why does an Azure VM need a NIC?
3. What is a private IP?
4. What is a public IP?
5. Can a VM have multiple NICs?
6. Can a NIC have multiple IP configurations?
7. Can you remove the primary NIC from a VM?
8. Why would you use a secondary NIC?

### NSG

9. What is an NSG?
10. What is an inbound rule?
11. What is an outbound rule?
12. What is NSG priority?
13. What happens when two rules conflict?
14. What is a Service Tag?
15. What is `VirtualNetwork`?
16. What is `Internet`?
17. What is an Application Security Group?
18. Does an ASG itself contain security rules?

### Architecture

19. Where can an NSG be associated?

```text
NIC
Subnet
```

20. What is the difference between NIC-level and subnet-level NSG?

21. If an NSG is associated with a subnet, which VMs are affected?

22. If an NSG is associated with a NIC, which VM is affected?

23. Can a VM have one NSG at the NIC and another at the subnet?

24. If the subnet NSG allows port 80 but the NIC NSG denies port 80, what happens?

25. Why would you use an ASG instead of putting individual VM IP addresses into NSG rules?

---

# 🔥 Final Interview Challenge

You are asked:

> **"We have 100 VMs in a VNet. We want to allow Web servers to communicate with Application servers on port 8080, but Application servers should not be accessible from the Internet. How would you design the NSGs?"**

Your answer should include:

```text
VNet
  │
  ├── Web Subnet
  │     └── ASG-Web
  │
  ├── App Subnet
  │     └── ASG-App
  │
  └── DB Subnet
        └── ASG-DB
```

And explain:

**Internet → Web → App → Database**

with appropriate **NSG + ASG + Service Tag** rules.

This assignment gives you a very strong practical foundation before moving to **VNet Peering, Route Tables/UDR, NAT Gateway, Azure Load Balancer and Application Gateway**.
