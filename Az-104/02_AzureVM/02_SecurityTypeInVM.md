In Azure VM creation, **Security type** determines **how strongly Azure protects the VM during boot and while it is running**.

When you create a VM, you may see options such as:

1. **Standard**
2. **Trusted launch**
3. **Confidential virtual machines**

Think of them as increasing levels of VM security:

```text
Standard
   ↓
Trusted Launch
   ↓
Confidential VM
```

But they solve **different security problems**, so it's better to understand each one separately.

---

## 1. Standard

This is the traditional VM configuration.

The VM uses the normal Azure VM boot and security mechanisms without the additional Trusted Launch protections.

### Example

Suppose you create:

```text
VM: WebServer01
OS: Ubuntu
Security type: Standard
```

You install:

```text
Apache
PHP
Your application
```

The VM works normally.

This is useful when:

* You need maximum compatibility
* Your workload doesn't require advanced VM security
* You are learning/testing Azure VMs
* You need features that may not yet support Trusted Launch

---

# 2. Trusted Launch

**Trusted Launch** provides additional protection against attacks that try to compromise the VM during the **boot process**.

It uses technologies such as:

* **Secure Boot**
* **vTPM (virtual Trusted Platform Module)**

### Simple example

Imagine someone tries to modify the VM's bootloader so that malicious code runs **before the operating system starts**.

With a traditional VM:

```text
VM starts
   ↓
Bootloader
   ↓
Operating System
```

Trusted Launch adds security checks:

```text
VM starts
   ↓
Secure Boot
   ↓
Verify trusted boot components
   ↓
Bootloader
   ↓
Operating System
```

If the boot components have been tampered with, the VM can detect the problem.

### vTPM

Trusted Launch also provides a **virtual TPM**.

Think of vTPM as a secure virtual hardware component that can store and protect cryptographic information and help establish the VM's boot integrity.

So:

```text
Trusted Launch
     |
     +-- Secure Boot
     |
     +-- vTPM
     |
     └-- Boot integrity protection
```

### Example

You have a Windows Server VM containing sensitive business applications:

```text
Windows Server
      +
Trusted Launch
      ↓
Secure Boot + vTPM
      ↓
Better protection against boot-level attacks
```

---

# 3. Confidential Virtual Machine

A **Confidential VM (CVM)** provides stronger protection for **data while it is being processed in memory**.

This is called:

> **Data in use protection**

Normally:

```text
Storage
   ↓
Data at rest

Network
   ↓
Data in transit

RAM
   ↓
Data in use
```

Encryption traditionally protects:

* Data at rest
* Data in transit

Confidential computing focuses especially on:

* **Data in use**

### Example

Suppose your application processes:

```text
Customer financial information
        ↓
Application
        ↓
RAM
```

The sensitive information has to be in memory while the application is processing it.

A Confidential VM uses hardware-based confidential computing technologies to help protect that memory from unauthorized access, including from certain privileged infrastructure-level threats.

Conceptually:

```text
Application
    ↓
Protected memory
    ↓
Sensitive data
```

---

# Standard vs Trusted Launch vs Confidential VM

| Feature                               | Standard | Trusted Launch | Confidential VM             |
| ------------------------------------- | -------- | -------------- | --------------------------- |
| Normal VM security                    | ✅        | ✅              | ✅                           |
| Secure Boot                           | ❌        | ✅              | Depends on configuration    |
| vTPM                                  | ❌        | ✅              | Supported capabilities vary |
| Protection against boot-level attacks | Basic    | **Strong**     | Strong                      |
| Protection of data in memory          | Basic    | Basic          | **Strong**                  |
| Performance/compatibility             | Highest  | High           | More restrictions           |
| Use for normal workloads              | ✅        | ✅              | Sometimes                   |
| Sensitive workloads                   | ⚠️       | ✅              | **✅ Best suited**           |

---

## A real-world example

Imagine a bank running three types of workloads.

### Web server

```text
Internet
   ↓
Web Server
```

It may use:

**Standard** or **Trusted Launch**

---

### Important application server

```text
Users
  ↓
Application Server
  ↓
Database
```

The company wants protection against boot-level attacks.

Use:

**Trusted Launch**

---

### Highly sensitive financial processing

```text
Customer data
      ↓
Financial application
      ↓
Sensitive data in RAM
```

The company wants additional protection for data while it is being processed.

Use:

**Confidential VM**

---

# Very important for your AZ-104 learning

Remember these three keywords:

### Standard

> **Normal VM**

### Trusted Launch

> **Protect the VM's boot process**

Think:

**Secure Boot + vTPM**

### Confidential VM

> **Protect sensitive data while it is being processed**

Think:

**Memory / data in use**

And this connects directly to the hibernation question you asked earlier:

> **Linux Confidential VMs and Linux Trusted Launch VMs currently don't support hibernation.**

So if you're practicing **Linux VM hibernation**, selecting **Standard** security type is the appropriate choice when the other VM requirements are also supported.
