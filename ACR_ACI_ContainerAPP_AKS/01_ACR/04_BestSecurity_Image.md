# Azure Container Registry — Image Security Best Practices

When we store container images in **Azure Container Registry (ACR)**, securing the registry is only one part of the job. We also need to make sure that the **image itself is trusted, up-to-date, minimally privileged, and free from known vulnerabilities**.

A good production flow is:

```text
Developer
    |
    v
Source Code
    |
    v
Docker Build
    |
    v
Security Scan
    |
    +---- Vulnerable? ----> Fix / Rebuild
    |
    v
ACR
    |
    v
Approved Image
    |
    v
AKS / VM / Container Apps
```

---

# 1. Use Trusted Base Images

Don't randomly download a base image from an unknown registry.

For example, avoid:

```dockerfile
FROM randomuser/ubuntu
```

Prefer trusted, maintained images.

For example:

```dockerfile
FROM nginx:alpine
```

or an official Microsoft image where appropriate.

The principle is:

```text
Trusted source
      ↓
Maintained image
      ↓
Security updates
      ↓
Your application
```

### Why?

Your application inherits vulnerabilities from its base image.

For example:

```text
Your Application
      |
      +-- Node.js
      |
      +-- Ubuntu
      |
      +-- OpenSSL
      |
      +-- System Libraries
```

A vulnerability in any of those components can affect your container.

---

# 2. Use Minimal Base Images

Don't use a huge operating-system image when you only need a small runtime.

For example:

```dockerfile
FROM ubuntu:latest
```

might contain many packages you don't need.

Instead, where appropriate:

```dockerfile
FROM nginx:alpine
```

or use a minimal runtime/distroless image where your application supports it.

Think:

```text
Large Image
   |
   +-- Many packages
   +-- More dependencies
   +-- Larger attack surface
```

versus:

```text
Minimal Image
   |
   +-- Required runtime
   +-- Required libraries
   +-- Application
```

### Rule

> **The smaller the image, the smaller the potential attack surface.**

---

# 3. Don't Use `latest` in Production

This is one of the most important practices.

Avoid:

```bash
docker pull myreg07.azurecr.io/mynginx:latest
```

for production deployments.

Why?

Because `latest` can point to different image versions over time.

Today:

```text
latest → v1
```

Tomorrow:

```text
latest → v2
```

Your deployment is no longer deterministic.

Instead use:

```text
mynginx:v1.2.3
```

or, even better, an immutable digest:

```text
mynginx@sha256:abc123...
```

### Recommended:

```text
Development:
latest

Testing:
v1.2.3

Production:
sha256:<digest>
```

---

# 4. Scan Images for Vulnerabilities

Before putting an image into production:

```text
Docker Image
      |
      v
Vulnerability Scanner
      |
      +---- Critical → STOP
      |
      +---- High → Investigate/Fix
      |
      +---- Medium → Review
      |
      v
Approved Image
```

In Azure environments, **Microsoft Defender for Cloud** can provide container security capabilities, including vulnerability assessment for container images in supported scenarios.

The important concept for students is:

> **Don't assume that an image is safe simply because it successfully built.**

A Docker build can succeed while the resulting image contains known vulnerabilities.

---

# 5. Scan the Base Image Too

Suppose:

```dockerfile
FROM ubuntu:22.04
```

Your application may have no vulnerabilities.

But:

```text
Application       → 0 vulnerabilities
Ubuntu base       → 5 vulnerabilities
```

The final image still contains those vulnerable components.

Therefore:

```text
Base Image
    ↓
Scan
    ↓
Application
    ↓
Scan final image
```

You should scan the **final image**, not only your source code.

---

# 6. Keep Images Updated

Vulnerabilities are continuously discovered.

For example:

```text
January
   ↓
Image v1

February
   ↓
New CVE discovered

March
   ↓
Updated base image

April
   ↓
Rebuild application
```

So don't build an image once and keep it forever.

Use a process such as:

```text
New vulnerability
       ↓
Update base image
       ↓
Rebuild
       ↓
Scan
       ↓
Push new image
       ↓
Redeploy
```

---

# 7. Never Put Secrets Inside the Image

This is a **critical** rule.

Never do this:

```dockerfile
ENV DB_PASSWORD="MyPassword123"
```

Don't put:

```text
Passwords
API keys
Connection strings
Certificates
Private keys
Tokens
```

inside your Dockerfile or image.

Why?

Anyone who obtains the image may potentially inspect its layers or configuration.

Instead:

```text
Container
    |
    | Runtime configuration
    v
Secret Store
```

In Azure, a common architecture is:

```text
Container
    |
    v
Managed Identity
    |
    v
Azure Key Vault
    |
    v
Secret
```

This is much better than:

```text
Container Image
    |
    +---- DB_PASSWORD ❌
    +---- API_KEY ❌
```

---

# 8. Use Managed Identity Instead of Credentials

This connects directly to the lab we just created.

We had:

```text
VM1
 |
 +-- User Assigned Managed Identity
          |
          +-- AcrPull
          |
          +-- ACR
```

That's preferable to putting an ACR username/password into a script.

Instead of:

```bash
docker login myreg07.azurecr.io \
  -u username \
  -p password
```

use Azure identity:

```bash
az login --identity
az acr login --name myreg07
```

This gives you:

```text
No hard-coded password
No stored registry credential
Managed identity
RBAC
Auditable access
```

---

# 9. Apply Least Privilege

Don't give every application:

```text
Owner
```

or:

```text
Contributor
```

to your ACR.

For example:

```text
AKS
 ↓
AcrPull
 ↓
ACR
```

If a CI/CD pipeline needs to push:

```text
CI/CD
 ↓
AcrPush
 ↓
ACR
```

And with repository-level ABAC:

```text
Frontend Pipeline
      |
      +-- Writer
      |
      v
   frontend

Backend Pipeline
      |
      +-- Writer
      |
      v
    backend
```

This is much better than:

```text
Everyone
   |
   +-- Owner
   |
   v
  ACR
```

---

# 10. Use Repository-Level Permissions When Appropriate

Since we just discussed:

**RBAC Registry + ABAC Repository Permissions**

this is an excellent image-security practice.

Suppose:

```text
myreg07
│
├── frontend
├── backend
├── payments
└── internal
```

Don't necessarily give the frontend team access to everything.

Instead:

```text
Frontend Team
      |
      +---- frontend ✅
      +---- backend  ❌
      +---- payments ❌
```

And:

```text
Payment Team
      |
      +---- payments ✅
      +---- frontend ❌
```

This reduces the blast radius if an identity is compromised.

---

# 11. Use Image Signing / Provenance

Another important concept is:

> **How do I know that this image was produced by my trusted build process?**

Imagine an attacker pushes:

```text
myreg07.azurecr.io/myapp:production
```

You don't want your production system blindly trusting it.

A stronger model is:

```text
Source Code
     ↓
Trusted CI/CD
     ↓
Build Image
     ↓
Sign / Attest
     ↓
ACR
     ↓
Verify
     ↓
Production
```

This helps establish **image provenance and integrity**.

Azure environments can use signing/provenance technologies such as **Notation** and OCI-based signing workflows.

---

# 12. Protect the ACR Network

Image security isn't only about the image.

You also need to protect the registry.

For sensitive production environments, consider:

```text
                Azure VNet
                   |
                   v
              Private Endpoint
                   |
                   v
                  ACR
```

Instead of allowing unrestricted public network access:

```text
Internet
   |
   v
ACR
```

you can design:

```text
Private Network
      |
      v
Private Endpoint
      |
      v
ACR
```

This reduces exposure of the registry endpoint.

---

# 13. Enable Appropriate Defender / Security Monitoring

For production environments, use Azure security services to monitor container-related risks.

Conceptually:

```text
                 Azure
                   |
          +--------+--------+
          |                 |
          v                 v
         ACR        Defender for Cloud
          |                 |
          +--------+--------+
                   |
                   v
             Security Findings
```

You can use security recommendations and vulnerability information to identify images/components that require remediation.

---

# 14. Don't Run Containers as Root

Another major container security practice.

Avoid:

```dockerfile
USER root
```

when it isn't required.

Prefer a non-root user.

For example:

```dockerfile
FROM nginx:alpine

USER nginx
```

The exact user depends on the base image.

Why?

If an attacker compromises the application:

```text
Root container
     ↓
Potentially greater impact
```

versus:

```text
Non-root container
     ↓
Reduced privileges
     ↓
Smaller blast radius
```

---

# 15. Use Read-Only Filesystems Where Possible

If your application doesn't need to modify its filesystem, consider running it with a read-only root filesystem.

Conceptually:

```text
Container
 |
 +-- Application files → Read Only
 |
 +-- Temporary data → Specific writable location
```

This can make certain attacks harder.

---

# 16. Don't Install Unnecessary Packages

Avoid Dockerfiles like:

```dockerfile
RUN apt-get update
RUN apt-get install curl
RUN apt-get install vim
RUN apt-get install wget
RUN apt-get install git
RUN apt-get install ...
```

if these aren't required by the application.

Every additional package potentially increases the attack surface.

Better:

```text
Required dependencies only
        ↓
Smaller image
        ↓
Fewer vulnerabilities
```

---

# 17. Use Multi-Stage Docker Builds

This is particularly useful for compiled applications.

Instead of putting the compiler and build tools into the production image:

```dockerfile
FROM node:latest AS build

# Build application


FROM nginx:alpine

# Only copy production artifacts
COPY --from=build /app/dist /usr/share/nginx/html
```

Architecture:

```text
              Build Image
           +--------------+
           | Compiler     |
           | SDK          |
           | Git          |
           | Build tools  |
           +------+-------+
                  |
                  | Build
                  v
           Production Image
           +--------------+
           | Application  |
           | Runtime      |
           +--------------+
```

The production image is much smaller.

---

# 18. Use `.dockerignore`

Don't accidentally copy unnecessary or sensitive files into your image.

Create:

```text id="z6k4y7"
.dockerignore
```

Example:

```text id="7ayjtp"
.git
.env
node_modules
*.log
README.md
```

Then:

```dockerfile
COPY . .
```

doesn't blindly copy everything from the build context.

Especially important:

```text
.env
```

because it may contain secrets.

---

# 19. Don't Trust Image Tags Alone

Remember:

```text
myapp:v1
```

is a mutable reference.

A tag can potentially be moved to another image.

For stronger deployment integrity:

```text
myapp@sha256:abc123...
```

The digest identifies the specific image content.

Production deployment can therefore be:

```text
Production
    |
    v
myapp@sha256:xxxxxxxx
```

rather than:

```text
myapp:latest
```

---

# 20. Implement Image Lifecycle Management

Don't allow thousands of old images to accumulate indefinitely.

For example:

```text
ACR
│
├── v1
├── v2
├── v3
├── v4
├── ...
└── v500
```

You can establish retention policies and cleanup processes appropriate to your environment.

For example:

```text
Keep:
Latest 10 production versions

Delete:
Old development images
Unused tags
Expired builds
```

Be careful not to delete images still referenced by production workloads.

---

# Production ACR Security Architecture

Putting everything together:

```text
                         Developer
                             |
                             v
                       Source Control
                             |
                             v
                         CI/CD Build
                             |
                     +-------+-------+
                     |               |
                     v               v
                Vulnerability     Secret Scan
                   Scan
                     |
                     v
                Image Build
                     |
                     v
              Image Signing /
                 Provenance
                     |
                     v
              +---------------+
              |      ACR      |
              |               |
              | Private       |
              | Repository    |
              | Permissions   |
              +-------+-------+
                      |
              Private Endpoint
                      |
                      v
                 Azure VNet
                      |
          +-----------+-----------+
          |                       |
          v                       v
         AKS                     VM
          |                       |
      Managed Identity       Managed Identity
          |                       |
       AcrPull                  AcrPull
          |                       |
          +-----------+-----------+
                      |
                      v
                 Container
```

---

# ⭐ The 10 Rules I Would Teach Your Students

If you want a simple exam/interview checklist:

```text
1. Use trusted base images
2. Use minimal images
3. Scan images for vulnerabilities
4. Keep base images and dependencies updated
5. Never store secrets in images
6. Use Managed Identity + RBAC
7. Use least-privilege repository permissions
8. Use image signing/provenance where appropriate
9. Don't use latest for production
10. Run containers as non-root
```

And the overall security pipeline is:

```text
        BUILD
          ↓
     Minimal Image
          ↓
    Vulnerability Scan
          ↓
      Secret Scan
          ↓
     Sign / Attest
          ↓
          ACR
          ↓
   RBAC / ABAC Access
          ↓
    Private Networking
          ↓
     AKS / VM / Apps
          ↓
       Runtime
       Security
```

### One key distinction

**ACR security** and **container image security** are related but different:

```text
ACR Security
├── Authentication
├── RBAC / ABAC
├── Private Endpoint
├── Network access
└── Registry protection

Image Security
├── Trusted base image
├── Vulnerability scanning
├── Minimal image
├── No secrets
├── Signing/provenance
├── Non-root
└── Immutable deployment
```

That distinction is very useful when you're teaching **ACR → Docker Images → VM/AKS** as one complete module.
