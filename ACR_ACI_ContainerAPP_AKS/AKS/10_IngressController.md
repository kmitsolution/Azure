## What is Ingress in Kubernetes?

An **Ingress** is an API object that manages **external HTTP/HTTPS access** to services inside the cluster.

👉 Instead of exposing each Service separately (NodePort / LoadBalancer), Ingress provides:

* **Single entry point**
* **Routing rules (based on path or domain)**
* **Centralized control**

---

### Basic Idea

```
Internet → Ingress → Services → Pods
```

* Ingress receives requests
* Based on rules, it forwards traffic to the correct Service

---

## Important: Ingress vs Ingress Controller

👉 **Ingress (resource)** = just rules (YAML definition)
👉 **Ingress Controller** = actual component that implements those rules

Without a controller → Ingress does nothing ❗

---

## NGINX Ingress Controller

The **NGINX Ingress Controller** is one of the most commonly used controllers.

👉 It:

* Runs as Pods inside the cluster
* Uses **NGINX** as a reverse proxy
* Watches Ingress resources
* Routes traffic accordingly

---

## Architecture Overview

```
Internet
   |
NodePort / LoadBalancer
   |
NGINX Ingress Controller
   |
-------------------------
|           |           |
nginx svc   tomcat svc  other svc
```

---

## Step-by-Step Explanation of Your Example

### 1. Install NGINX Ingress Controller

You applied:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/baremetal/deploy.yaml
```

👉 This:

* Creates a namespace: `ingress-nginx`
* Deploys:

  * Controller Pods
  * Service (usually NodePort in bare metal)

---

### 2. Verify Namespace and Services

```bash
kubectl get ns
kubectl get svc -n ingress-nginx
```

👉 You will see:

* A Service exposing the Ingress Controller
* Often of type **NodePort**

---

## 3. Create Backend Applications

### NGINX Pod + Service

* Pod: nginx (port 80)
* Service: ClusterIP → `nginx`

### Tomcat Pod + Service

* Pod: tomcat (port 8080)
* Service: ClusterIP → `tomcat`

👉 These services are **internal only**

---

## 4. Create Ingress Resource

Your YAML:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /nginx
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: tomcat
            port:
              number: 8080
```

---

## How This Ingress Works

### Routing Logic

| URL Path | Forwarded To Service |
| -------- | -------------------- |
| `/nginx` | nginx service        |
| `/web`   | tomcat service       |

---

### Traffic Flow

```
User → NodeIP:NodePort
        ↓
NGINX Ingress Controller
        ↓
Path-based routing
   ├── /nginx → nginx service → nginx pod
   └── /web   → tomcat service → tomcat pod
```

---

## Role of Annotation

```yaml
nginx.ingress.kubernetes.io/rewrite-target: /
```

👉 This tells NGINX:

* Remove `/nginx` or `/web` from URL
* Forward request as `/` to backend

Example:

```
/nginx → /
/web → /
```

---

## How You Access It

1. Find NodePort of ingress controller:

```bash
kubectl get svc -n ingress-nginx
```

2. Access using:

```
http://NodeIP:NodePort/nginx
http://NodeIP:NodePort/web
```

---

## Key Concepts to Remember

### 1. Ingress does NOT expose directly

* It needs an **Ingress Controller**

### 2. Works only for HTTP/HTTPS

* Not for TCP/UDP (by default)

### 3. Uses rules

* Path-based routing (`/nginx`, `/web`)
* Can also use **host-based routing**

---

## Why Use Ingress?

Without Ingress:

* Each Service needs NodePort / LoadBalancer ❌

With Ingress:

* Single entry point ✅
* Clean URLs ✅
* Centralized routing ✅

---

## Simple Summary

* You created:

  * 2 Pods (nginx, tomcat)
  * 2 ClusterIP Services
  * 1 Ingress resource
* NGINX Ingress Controller:

  * Receives external traffic
  * Routes based on path
* Result:

  * `/nginx` → nginx app
  * `/web` → tomcat app

---

If you want, I can next explain:
👉 **Ingress in AWS EKS using ALB (very important for real-world + interviews)**
