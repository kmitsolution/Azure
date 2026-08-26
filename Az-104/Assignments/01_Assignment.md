## Azure Basics – Assignment 1

### Topics Covered

You should use the following concepts:

* Azure Architecture
* Azure Regions
* Availability Zones
* Region Pairs
* Azure Subscriptions
* Resource Groups
* Azure Availability Sets
* Azure Spot Instances
* Azure Pricing Models
* Microsoft Entra ID

---

## Part 1 – Azure Architecture

### Task 1: Explain Azure Global Infrastructure

Answer the following:

1. What is an Azure Region?
2. What is an Availability Zone?
3. What is a Region Pair?
4. What is the difference between a Region and an Availability Zone?
5. Why does Azure have multiple regions?
6. What happens if an Azure region becomes unavailable?
7. What is the purpose of Azure's global infrastructure?

### Practical Task

Choose **one Azure region** and document:

* Region name
* Country
* Availability Zones, if supported
* Region pair
* Why you would choose this region for an application

---

# Part 2 – Subscription and Resource Groups

### Scenario

Your company is developing an application called **EmployeePortal**.

You have two environments:

* Development
* Production

The company wants proper separation of resources.

### Your Task

Design the following structure:

```text
Azure Tenant
│
├── Subscription
│   │
│   ├── Resource Group: EmployeePortal-Dev
│   │
│   └── Resource Group: EmployeePortal-Prod
│
└── Microsoft Entra ID
```

Answer:

1. What is an Azure Subscription?
2. Why would you separate Development and Production?
3. What is a Resource Group?
4. Can resources from different resource groups communicate with each other?
5. Can one resource group contain resources from different Azure regions?
6. What happens to resources when a Resource Group is deleted?
7. Who controls access to resources in the subscription?

### Practical Task

Using the Azure Portal:

* Create a Resource Group called:

```text
rg-employeeportal-dev
```

* Choose a region.
* Create another Resource Group:

```text
rg-employeeportal-prod
```

Do **not** create expensive resources just for this assignment.

---

# Part 3 – Availability Zone vs Availability Set

This is an important AZ-104 topic.

### Scenario

You are designing a web application that must remain available even if one physical datacenter fails.

You have three VMs:

```text
VM1
VM2
VM3
```

### Task

Design two architectures.

### Architecture A – Availability Set

```text
Region
│
└── Availability Set
      ├── VM1
      ├── VM2
      └── VM3
```

Explain:

* What problem does Availability Set solve?
* What are Fault Domains?
* What are Update Domains?
* Why are the VMs distributed?

### Architecture B – Availability Zones

```text
Azure Region
│
├── Zone 1 → VM1
├── Zone 2 → VM2
└── Zone 3 → VM3
```

Explain:

* Why is this architecture more resilient to datacenter-level failures?
* What is the difference between Availability Set and Availability Zone?
* When would you choose each one?

### Important Question

**If Availability Zones are available in a region, why might someone still use an Availability Set?**

---

# Part 4 – Spot VM Assignment

### Scenario

Your company wants to run a batch-processing application.

The application:

* Runs for several hours
* Can be stopped and restarted
* Does not require continuous availability
* Should cost as little as possible

You have two options:

```text
Normal VM
Spot VM
```

### Questions

1. Which one would you choose?
2. Why?
3. What is an Azure Spot VM?
4. Why can Azure evict a Spot VM?
5. What is the difference between:

```text
Capacity Only
```

and

```text
Price or Capacity
```

6. Give **three real-world applications** where Spot VMs are suitable.
7. Give **three applications** where Spot VMs should NOT be used.

### Practical Task

Create a Spot VM in Azure Portal.

Document:

* VM name
* Region
* VM size
* Eviction type
* Maximum price
* OS
* Why you selected those settings

You can delete the VM after completing the exercise.

---

# Part 5 – Azure Pricing Assignment

Your company gives you this requirement:

> "We need to run an application for one year. The application must be available continuously and should have predictable monthly costs."

Compare these pricing approaches:

| Pricing Model                     | Your Analysis |
| --------------------------------- | ------------- |
| Pay-As-You-Go                     | ?             |
| Reserved Instances / Reservations | ?             |
| Spot                              | ?             |

Answer:

1. Which pricing model would you recommend?
2. Why?
3. Which option provides the most flexibility?
4. Which option can provide significant savings when usage is predictable?
5. Why isn't Spot suitable for this requirement?

### Bonus

Find the estimated price of a small VM in your selected Azure region using the Azure Pricing Calculator and record:

```text
VM Size:
Region:
OS:
Estimated Monthly Cost:
Estimated Annual Cost:
```

---

# Part 6 – Microsoft Entra ID Assignment

### Scenario

Your company has the following employees:

```text
Ravi       → Developer
Priya      → Developer
John       → Administrator
David      → Manager
```

You need to implement access control.

Create the following groups:

```text
Developers
Administrators
Managers
```

Add users to the appropriate groups.

### Questions

1. What is Microsoft Entra ID?
2. What is a User?
3. What is a Group?
4. What is a Role?
5. What is RBAC?
6. What is the difference between authentication and authorization?
7. Why is assigning permissions to groups better than assigning permissions individually?

### Practical Task

In Microsoft Entra ID:

1. Create a test user.
2. Create a group called:

```text
Azure-Developers
```

3. Add the test user to the group.
4. Explain how you would give this group access to an Azure Resource Group.

---

# Part 7 – Real-World Case Study

This is the **main assignment**.

## Company: ABC Online Shopping

ABC is launching an e-commerce application.

Requirements:

### Application

The application consists of:

```text
Web Application
Database
Background Processing
```

### Requirements

1. The application will initially serve customers in India.
2. The application should be highly available.
3. Production and Development must be separated.
4. Development should have lower cost.
5. Background processing can tolerate VM eviction.
6. Production web servers must not depend on Spot VMs.
7. Only administrators should have production access.
8. Developers should have access to Development.
9. The company wants predictable costs for production.
10. The application should survive a failure affecting one datacenter.

---

## Your Design

Create an architecture similar to:

```text
                 Microsoft Entra ID
                        │
                        │
                 Azure Subscription
                        │
          ┌─────────────┴─────────────┐
          │                           │
     Development                  Production
          │                           │
     Resource Group              Resource Group
          │                           │
      ┌───┴────┐              ┌──────┴──────┐
      │         │              │             │
     VM       VM             Zone 1        Zone 2
                               VM             VM
                               
                         Background Worker
                              Spot VM
```

### You need to decide:

**1. Region**

Which Azure region will you use?

**2. Region Pair**

What is its region pair and why is that important?

**3. Availability**

Would you use:

* Availability Set
* Availability Zones
* Both

Explain why.

**4. Subscription**

Would you use:

* One subscription
* Multiple subscriptions

Explain your choice.

**5. Resource Groups**

Design your Resource Groups.

**6. Pricing**

Which pricing model would you use for:

* Production web servers
* Development VMs
* Background processing

**7. Identity**

Design:

```text
Administrators
Developers
Managers
```

and explain what access each group should receive.

---

# Part 8 – Interview Questions

Finally, answer these without looking at your notes.

### Q1

What is the difference between:

```text
Tenant
Subscription
Resource Group
Resource
```

---

### Q2

Explain this hierarchy:

```text
Microsoft Entra Tenant
        ↓
Subscription
        ↓
Resource Group
        ↓
Resource
```

---

### Q3

What is the difference between:

```text
Region
Availability Zone
Availability Set
Region Pair
```

---

### Q4

A company has a workload that can tolerate interruption. It wants the lowest possible VM cost.

**What would you recommend and why?**

---

### Q5

A production application cannot tolerate VM eviction.

**Would you use Spot VM? Why or why not?**

---

### Q6

Three VMs are placed in an Availability Set.

**What happens if one physical server fails?**

---

### Q7

Why would you deploy VMs across Availability Zones?

---

### Q8

What is the difference between:

**Authentication**

and

**Authorization?**

---

### Q9

A developer needs access only to the Development Resource Group.

Would you give the developer:

```text
Owner
Contributor
Reader
```

Which would you choose and why?

---

### Q10 – Design Question

Design an Azure environment for:

> **100 developers + 10 administrators + 1 production application + 1 development environment**

Your answer should include:

```text
Tenant
Subscriptions
Resource Groups
Regions
Availability
Identity
RBAC
Pricing
Spot VMs
```

---

## ⭐ Challenge Task

Don't just answer the questions.

Draw your own **Azure architecture diagram** on paper or using draw.io.

Your final diagram should contain:

```text
                    Azure
                      │
              Microsoft Entra ID
                      │
              ┌───────┴────────┐
              │                │
          Dev Subscription   Prod Subscription
              │                │
          Dev RG             Prod RG
              │                │
          ┌───┴───┐        ┌───┴────┐
          │       │        │        │
         VM      VM       Zone 1   Zone 2
                            │        │
                           VM       VM
                                   
                         Background Worker
                              Spot VM
```

Then explain **why you designed it this way**.

This will be much closer to an **AZ-104 real-world design exercise** than simply memorizing definitions.
