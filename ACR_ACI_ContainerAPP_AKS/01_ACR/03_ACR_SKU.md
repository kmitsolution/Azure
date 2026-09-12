 In **Azure Container Registry (ACR)**, **SKU** determines the level of registry capabilities, performance, storage, and advanced features available.

The three main SKUs are:

```text
ACR
│
├── Basic
├── Standard
└── Premium
```

A simple way to remember them:

> **Basic = Learning / Small workloads**
> **Standard = Production registry**
> **Premium = Enterprise / Advanced security & multi-region**

---

# 1. ACR Basic

**Basic** is intended for low-volume scenarios and learning/development.

Architecture:

```text
Developer
    │
    ▼
Basic ACR
    │
    ├── Images
    └── OCI Artifacts
```

### Good for

* Learning Azure
* Development
* Small projects
* Testing
* CI/CD experimentation
* Small image repositories

For example, your current lab:

```text
Dockerfile
    ↓
ACR Tasks
    ↓
myreg07
    ↓
mynginx:v1
    ↓
ACI
```

Basic can be perfectly adequate for this type of lab.

### Limitations

Basic doesn't provide the full set of advanced ACR capabilities available in Premium.

---

# 2. ACR Standard

**Standard** is designed for higher-volume workloads and production scenarios.

Architecture:

```text
Developers / CI-CD
       │
       ▼
   Standard ACR
       │
   ┌───┴────┐
   │        │
Images    Artifacts
   │
   ├── AKS
   ├── ACI
   └── App Service
```

Compared with Basic, Standard provides more capacity/performance.

Good for:

* Production applications
* More frequent image pulls/pushes
* CI/CD pipelines
* Multiple development teams
* Larger repositories

---

# 3. ACR Premium

This is where the major enterprise capabilities come in.

Architecture:

```text
                     Premium ACR
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
    Geo-replication   Private Link    Security
          │              │              │
          ▼              ▼              ▼
      Region 1        Private IP     Advanced
      Region 2        VNet           controls
      Region 3
```

Premium is intended for enterprise-scale requirements.

It provides advanced capabilities such as:

* **Geo-replication**
* **Private endpoints**
* **Zone redundancy** where supported
* **Customer-managed keys** where supported
* Advanced repository/security capabilities
* Higher throughput/capacity
* Large-scale enterprise registry scenarios

The exact feature availability can vary by Azure region and evolve over time, so for production architecture you should verify the current regional feature matrix.

---

# 4. Basic vs Standard vs Premium

A useful high-level comparison:

| Feature                          | Basic                 | Standard              | Premium                   |
| -------------------------------- | --------------------- | --------------------- | ------------------------- |
| Private container registry       | ✅                     | ✅                     | ✅                         |
| Docker/OCI images                | ✅                     | ✅                     | ✅                         |
| ACR Tasks                        | ✅                     | ✅                     | ✅                         |
| RBAC                             | ✅                     | ✅                     | ✅                         |
| Repository capabilities          | ✅                     | ✅                     | ✅                         |
| Higher throughput                | —                     | ✅                     | ✅                         |
| Private Endpoint                 | ❌                     | ❌                     | ✅                         |
| Geo-replication                  | ❌                     | ❌                     | ✅                         |
| Zone redundancy                  | Limited/not available | Limited/not available | Supported where available |
| Customer-managed encryption keys | ❌                     | ❌                     | ✅                         |
| Enterprise workloads             | Limited               | ✅                     | ✅                         |
| Multi-region registry            | ❌                     | ❌                     | ✅                         |

**Important:** Some individual features have prerequisites or regional limitations, so don't treat the table as a guarantee that every feature is available in every region.

---

# 5. Real-world example

Suppose you're a small developer.

```text
Developer
   │
   ▼
ACR Basic
   │
   ▼
ACI
```

Perfectly reasonable.

---

Now suppose you have:

```text
20 developers
       │
       ▼
ACR Standard
       │
       ├── Dev
       ├── Test
       └── Production
```

Standard may be appropriate.

---

Now imagine a global company:

```text
                    Premium ACR
                         │
             ┌───────────┼───────────┐
             │           │           │
             ▼           ▼           ▼
          Central     West US      Europe
           India
             │           │           │
             ▼           ▼           ▼
            AKS         AKS         AKS
```

Premium becomes much more attractive because **geo-replication** can place registry replicas closer to workloads in multiple regions.

---

# 6. Geo-replication — Why Premium matters

Suppose your application runs in:

```text
Central India
West Europe
East US
```

Without geo-replication:

```text
                    ACR
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       India       Europe       US
          │          │          │
       Long       Long         Long
       distance   distance     distance
```

With Premium geo-replication:

```text
                 Premium ACR
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
   Central India  West Europe   East US
      Replica       Replica      Replica
        │            │            │
        ▼            ▼            ▼
       AKS          AKS          AKS
```

The registry replicates the images to the selected Azure regions.

This can improve:

* Image pull performance
* Deployment speed
* Regional availability
* Resilience

---

# 7. Private Endpoint — Premium

Suppose you currently have:

```text
ACI/VM
   │
   │ Internet
   ▼
ACR
```

With Premium Private Endpoint:

```text
                 Azure VNet
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
        VM/AKS                Private
                               Endpoint
                                  │
                                  ▼
                                 ACR
```

The registry can be accessed through a private IP in your VNet rather than relying on public network access.

This is a very common enterprise architecture.

---

# 8. How to check your ACR SKU

For your registry `myreg07`:

```bash
az acr show \
  --name myreg07 \
  --query sku.name \
  -o tsv
```

You might get:

```text
Standard
```

Or:

```text
Basic
```

or:

```text
Premium
```

You can also see more information:

```bash
az acr show \
  --name myreg07 \
  --query "{name:name,sku:sku.name,location:location,loginServer:loginServer}" \
  -o table
```

---

# 9. Change SKU

One nice thing about ACR is that you can change the SKU.

For example:

```bash
az acr update \
  --name myreg07 \
  --sku Premium
```

Or:

```bash
az acr update \
  --name myreg07 \
  --sku Standard
```

The available upgrade/downgrade paths and feature impacts should be checked before doing this on production resources.

---

# 10. Portal

You can check it through:

**Azure Portal → Container Registries → myreg07 → Properties**

Look for:

```text
SKU
```

You'll see something like:

```text
Standard
```

You can also find SKU-related configuration under the registry's properties/settings depending on the current Portal UI.

---

# 11. Which SKU should you use?

For **your current Azure training labs**:

### Learning / AZ-104 lab

```text
Basic
```

is usually enough.

### Small production application

```text
Standard
```

is a reasonable choice.

### Enterprise application

If you need things like:

```text
Private Endpoint
Geo-replication
Enterprise-scale registry
Advanced security features
```

then:

```text
Premium
```

is the appropriate tier to investigate.

---

# 12. Easy interview answer

If the interviewer asks:

**"What is SKU in Azure Container Registry?"**

You can say:

> **SKU defines the service tier of Azure Container Registry. ACR provides Basic, Standard, and Premium tiers. Basic is suitable for development and low-volume workloads, Standard provides higher capacity and throughput for production workloads, and Premium adds advanced enterprise capabilities such as geo-replication and private endpoints.**

### Remember it this way:

```text
       ACR SKU
          │
 ┌────────┼─────────┐
 ▼        ▼         ▼
Basic   Standard   Premium
 │        │          │
Lab     Production  Enterprise
                    │
              ┌─────┴─────┐
              ▼           ▼
          Geo-rep       Private
                        Endpoint
```

**One important point:** SKU is different from the **DNL scope** we just discussed. `SKU` determines the **service tier/capabilities**, while `DNL scope` controls **registry DNS-name reuse/protection**.
