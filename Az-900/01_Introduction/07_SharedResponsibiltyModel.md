# Shared Responsibility Model in Azure — AZ-900

## 1. What is the Shared Responsibility Model?

The Shared Responsibility Model explains how security and management responsibilities are divided between the cloud service provider (Microsoft Azure) and the customer.

In traditional on-premises infrastructure, the organization is responsible for managing almost everything, including physical servers, networking, operating systems, applications and data security.

When an organization moves to Azure, Microsoft takes responsibility for the underlying cloud infrastructure, while the customer remains responsible for certain aspects of their workloads. The exact division depends on which cloud service model is being used.

Key concept for AZ-900: Microsoft is responsible for the security of the cloud, while customers are responsible for the security in the cloud, with the precise boundaries varying by service.

## 2. Understanding responsibility by cloud model


On-premises infrastructure

The organization manages the physical infrastructure and the entire software stack, including networking, servers, operating systems, applications, identity and data.

[Cloud, Web, DevOps: Azure Virtual Machine – Best Practices](https://images.openai.com/static-rsc-4/k7O0T-8TQ_T2YMCjvlmJTxkOessGslb5vaZGaA35L_wz3PxmsCUrOeP6lZ57ZftMd35MYnS37FADZyW1YlRzS5f9aZNhnU5TtO3AwfFTqB4tNzwtJHOQzuaZjrJIKkJNu1g56kX26Tw8nIGocEE2gwy4sF-WTwwjX5iYJhN4SQY?purpose=inline)

IaaS — Infrastructure as a Service

Example: Azure Virtual Machines.

Microsoft manages the physical datacenter, physical network and host infrastructure. The customer typically manages the guest operating system, applications, data, identities and VM-level configurations.

[What is PaaS (Platform-as-a-Service)? ❓ Guide by Wallarm](https://images.openai.com/static-rsc-4/abw-bfTIT0-Qt2OKtq_AXqEU1Ec5pp_ic6AyYWr4nDxs1Ct8yl4Hp0XiczMheIohFI4yAIMR_Wv1lSlIvw7mpKTSWINhjzKDgZmY8MsJke6yRR_Q7c1UWeJxLMBJfow4s0zSDxC1WNds9p_1sZ7DtKrlDfXf-NPyCSXju7Byj58?purpose=inline)

PaaS — Platform as a Service

Example: Azure App Service.

Microsoft manages the infrastructure and much of the platform, including the operating system and runtime environment. The customer focuses on the application, data, identities and application configuration.

[9 Practical SaaS Email Marketing Campaigns](https://images.openai.com/static-rsc-4/3IedJ3Y-62Mx73pH-78wbRE9RGAjqihtW9Mb1A9CSDqgbNdpeSazdND-m9RZc4ZGCVWlq3L8uAdeoCjoPzEeG3GUbl1aNV2_65ubEEJFEnr5YFHP6ec1lYSUNqvsSgoXpYPICusQMiILJneXfXUsEtPHezfncVPXaLsblfNbmgw?purpose=inline)

SaaS — Software as a Service

Example: Microsoft 365.

The provider manages most of the application stack. Customers still manage responsibilities such as their users, access permissions, data handling and appropriate security settings.

## 3. Responsibility comparison

The following table shows the typical division of responsibilities. Exact boundaries can vary by service and configuration.

| Responsibility                | On-premises | IaaS      | PaaS      | SaaS                              |
| ----------------------------- | ----------- | --------- | --------- | --------------------------------- |
| Physical datacenter           | Customer    | Microsoft | Microsoft | Microsoft                         |
| Physical networking           | Customer    | Microsoft | Microsoft | Microsoft                         |
| Physical hosts                | Customer    | Microsoft | Microsoft | Microsoft                         |
| Operating system              | Customer    | Customer  | Microsoft | Microsoft                         |
| Runtime and middleware        | Customer    | Customer  | Microsoft | Microsoft                         |
| Application                   | Customer    | Customer  | Customer  | Microsoft                         |
| Data and access configuration | Customer    | Customer  | Customer  | Shared, with customer obligations |

For SaaS, customers still have important responsibilities for their data, accounts, permissions and usage. Microsoft manages the application infrastructure, but that does not mean every security setting is automatically configured correctly.

## 4. Real-world example: Hosting an application on Azure

Imagine a company called ABC Retail wants to host its online shopping application.

### Scenario A — Azure Virtual Machine (IaaS)

The company creates an Azure VM and installs a web server and application.

Microsoft is responsible for:

- Physical servers and datacenter facilities.
- Physical networking.
- The underlying host infrastructure.
- Maintaining the cloud platform infrastructure.

ABC Retail is responsible for:

- Configuring and updating the guest operating system.
- Installing application updates.
- Securing application code and data.
- Configuring user access and permissions.
- Configuring the VM and network security controls appropriately.

If ABC Retail fails to patch a vulnerable guest operating system, that is generally the customer's responsibility.

### Scenario B — Azure App Service (PaaS)

The company deploys its web application to Azure App Service.

Microsoft manages the underlying infrastructure and platform components, including the operating system and managed runtime environment.

ABC Retail remains responsible for its application code, application-level vulnerabilities, data and access configuration.

The company does not need to administer the underlying operating system in the same way it would with a VM.

### Scenario C — Microsoft 365 (SaaS)

The company uses Microsoft 365 for email and collaboration.

Microsoft manages the hosted application and its underlying infrastructure. ABC Retail still needs to manage users, permissions, authentication settings and how employees handle company data.

Conclusion: As we move from IaaS to PaaS to SaaS, the provider generally takes on more responsibility for the underlying technology stack, reducing the customer's operational burden.

## 5. What about security of the cloud vs. security in the cloud?

This is an important distinction for AZ-900.

### Security of the cloud

Microsoft is responsible for protecting the underlying cloud infrastructure, such as physical datacenters, physical hosts and core infrastructure.

### Security in the cloud

Customers are responsible for securing their workloads according to the service model, including data, identities, permissions, applications and customer-managed configurations.

## 6. Important AZ-900 exam points

- The cloud provider is responsible for physical datacenters and underlying physical infrastructure.
- In IaaS, the customer generally manages the guest operating system.
- In PaaS, the provider manages the operating system and platform; the customer manages the application and data.
- In SaaS, the provider manages most of the application stack, but the customer retains responsibilities for users, access and data.
- Customer responsibility does not disappear when moving to the cloud.
- Shared responsibility varies according to the service and configuration.

Official reference: [Microsoft Learn — Shared responsibility in the cloud](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility?utm_source=chatgpt.com)

## 7. AZ-900 practice questions

<b>1\. Who is responsible for maintaining the physical datacenter hosting Azure services?</b>

A. The customer

B. Microsoft Azure

C. The application developer only

D. The internet service provider

<b>2\. A company runs an application on an Azure Virtual Machine. Who is generally responsible for patching the guest operating system?</b>

A. Microsoft in every case

B. The customer

C. The physical datacenter operator

D. The internet service provider

<b>3\. Which service model generally gives the customer the least responsibility for managing the underlying software stack?</b>

A. On-premises

B. IaaS

C. PaaS

D. SaaS

<b>4\. In Azure App Service, who is generally responsible for application code and application-level vulnerabilities?</b>

A. Microsoft alone

B. The customer

C. The physical hardware vendor

D. No one

<b>5\. Which statement best describes the Shared Responsibility Model?</b>

A. Microsoft is responsible for every security setting

B. The customer is responsible for the entire physical datacenter

C. Responsibilities are divided between provider and customer depending on the service

D. Cloud services do not require security management

Reset quiz

### Quick revision

Remember this simple pattern:

- IaaS: Microsoft manages the physical infrastructure; the customer generally manages the guest OS and application.
- PaaS: Microsoft manages the infrastructure and platform; the customer manages the application and data.
- SaaS: Microsoft manages most of the application stack; the customer still manages appropriate access, data and usage settings.
