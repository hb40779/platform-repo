# GitOps Platform Mini Tutorial

This repository set demonstrates how to build a **local, production‑grade GitOps platform** using **Kubernetes, Helm and ArgoCD**, following modern **platform engineering** principles.

The goal is not just to deploy an application, but to design a **scalable, reusable, multi‑environment GitOps setup** that mirrors real‑world practices.

---

## Project Goals

- Run everything **locally** (no paid cloud resources)
- Use **Git as the single source of truth**
- Separate **platform responsibilities** from **application responsibilities**
- Support **multiple applications** and **multiple environments**
- Use **ArgoCD multi‑source applications**
- Deploy applications using **values.yaml only** (no app Helm charts)

This project is intended as:
- a learning reference
- a portfolio project
- a lightweight tutorial for GitOps fundamentals

---

## Architecture Overview

```
App Repos (values only)
┌─────────────────────┐
│ hello-world-app     │
│ api-service-app     │
└─────────┬───────────┘
          │ values.yaml
          │
┌─────────▼───────────┐
│   ArgoCD            │
│  (multi-source)     │
└─────────┬───────────┘
          │
┌─────────▼───────────┐
│ Platform Repo       │
│ Helm app-template   │
│ Env definitions     │
└─────────┬───────────┘
          │
┌─────────▼───────────┐
│ Kubernetes Cluster  │
│ local / staging /   │
│ prod namespaces     │
└─────────────────────┘
```

---

## 🧱 Repositories

### 1️⃣ Platform Repository

**Responsibilities:**
- Helm application template
- ArgoCD applications
- Environment definitions
- GitOps orchestration

**Contains:**
- reusable Helm chart (`app-template`)
- `clusters/` folder with env definitions
- ArgoCD app‑of‑apps pattern

Platform repo never contains app‑specific configuration.

---

### 2️⃣ Application Repositories

Example:
- `hello-world-app`
- `api-service-app`

**Responsibilities:**
- application configuration only
- no Kubernetes manifests
- no Helm templates

Each app repo contains:
```text
values.yaml
```

This enforces clean ownership and independence.

---

## 💻 Local Environment Setup

### 1️⃣ Virtual Machine

- OS: Ubuntu Server 22.04 LTS
- Resources:
  - 2 CPU
  - 4 GB RAM

---

### 2️⃣ Install Container Runtime

```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
```

---

### 3️⃣ Kubernetes (k3s)

```bash
curl -sfL https://get.k3s.io | sh -
```

Configure kubectl:
```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

---

### 4️⃣ Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

### 5️⃣ ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access UI (port‑forward):
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

---

## 📦 Helm App Template

The platform provides a **single reusable Helm chart**:

Features:
- Deployment
- Service
- Optional Ingress
- Resource limits
- Probes
- Standardized naming

Applications never modify this chart.

---

## 🌍 Environments

Supported environments:

| Environment | Purpose |
|-----------|--------|
| local | development |
| staging | pre‑production |
| prod | production |

Each environment has:
- its own ArgoCD root application
- its own namespace(s)
- its own lifecycle rules

---

## 🔁 App‑of‑Apps Pattern

Each environment defines a root ArgoCD application:

```text
clusters/<env>/argocd/app-of-apps.yaml
```

This root app discovers all application definitions in:
```text
clusters/<env>/argocd/apps/
```

---

## 🔗 ArgoCD Multi‑Source Applications

Each application is defined as a **multi‑source ArgoCD Application**:

- Source 1: platform repo (Helm chart)
- Source 2: app repo (values.yaml)

This allows:
- auto‑deploy on app changes
- no dummy commits
- clean separation of ownership

Example:
```yaml
sources:
  - repoURL: platform-repo
    path: helm/app-template
    helm:
      valueFiles:
        - $values/values.yaml

  - repoURL: app-repo
    ref: values
```

---

## 🚀 Deploying Applications

To deploy a new application:

1. Create a new app repo with `values.yaml`
2. Add a new ArgoCD Application definition per environment
3. Commit and push

No Helm chart duplication.
No platform changes.

---

## 📈 Scaling to Multiple Applications

This platform supports:
- multiple apps
- multiple environments
- independent deploy cycles

Example applications:
- `hello-world`
- `api-service`

Both use the **same Helm chart**.

---

## 🧠 Key Learnings

- GitOps reacts to **state**, not commit events
- Helm charts are **platform contracts**
- values.yaml defines application intent
- ArgoCD multi‑source enables true app autonomy
- clean ownership prevents operational conflicts

---

## 📌 Next Steps

Possible extensions:
- promotion workflow (staging → prod)
- secret management (External Secrets / SOPS)
- HPA and autoscaling
- progressive delivery (Argo Rollouts)

---

## 🏁 Final Note

This project intentionally mirrors **real production design**, not tutorials.

If you understand and can explain this setup, you understand **modern GitOps and platform engineering fundamentals**.


