
# 🛍️ Style Store – Kubernetes + ArgoCD Microservices Demo

This is a demo project showcasing a microservices-based architecture for a fictional e-commerce platform "Style Store", deployed to Kubernetes using GitOps via ArgoCD.

---

## 🧱 Architecture Overview

- **Frontend** – React Single Page App served via NGINX (internal routing to backend)
- **Auth Service** – FastAPI app, uses PostgreSQL
- **Orders Service** – FastAPI app, uses PostgreSQL
- **PostgreSQL** – Single shared pod hosting two logical databases: `auth_db`, `orders_db`
- **ArgoCD** – For GitOps-based continuous delivery
- **SonarQube** *(optional)* – For code quality checks
- ❌ **Ingress** – Not used; NGINX inside the frontend handles all routing

---

## 📂 Project Structure

```
style-store-gitops/
├── apps/
│   ├── backend/      # FastAPI backend services: auth, orders
│   └── frontend/     # React + NGINX-based frontend app
├── infra/
│   ├── argo/         # ArgoCD Application manifests
│   └── K8s/          # Kubernetes deployments, services, jobs
└── README.md         # This documentation file
```

---

## 🔀 Routing – Inside Frontend NGINX (Not Ingress)

| Path             | Routed To         | Kubernetes DNS                            | Port |
|------------------|-------------------|-------------------------------------------|------|
| `/auth/**`       | `auth-service`    | `auth-service.default.svc.cluster.local`  | 8000 |
| `/api/orders/**` | `orders-service`  | `orders-service.default.svc.cluster.local`| 8000 |
| `/`              | Static frontend   | Served by NGINX container                 | 80   |

> All routing is handled via `nginx.conf` **inside the frontend Docker image**. No Kubernetes Ingress resource is applied.

---

## 🚀 Getting Started

### 1. Clone the GitOps Repo

```bash
git clone https://github.com/de2pakchauhan/style-store-gitops.git
cd style-store-gitops
```

### 2. Start Minikube

```bash
minikube start
```

### 3. Deploy ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Access ArgoCD UI:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Log in:

```bash
argocd login localhost:8080
```

### 4. Create ArgoCD App (App of Apps)

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: style-store
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/de2pakchauhan/style-store-gitops
    targetRevision: HEAD
    path: overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

Apply:

```bash
kubectl apply -f infra/argo/app-of-apps.yaml
```

### 5. Initialize Databases

```bash
kubectl apply -f apps/init-db/init-job.yaml
```

Creates `auth_db` and `orders_db` inside the shared PostgreSQL pod.

---

## 🌐 Accessing the App

Port forward the frontend service:

```bash
kubectl port-forward svc/frontend-service 8080:80
```

Then open:

```
http://localhost:8080
```

---

## 🧪 Verifying Setup

```bash
kubectl get pods
kubectl get svc
kubectl get events
```

You should see:

- `frontend`, `auth`, `orders`, and `postgres` pods
- Internal communication working via DNS
- Frontend proxying correctly to backend

---

## 🧹 Optional: SonarQube + CI

- Deploy SonarQube on a pod or external instance
- Add GitHub repo as a project
- Use SonarScanner to scan code during CI
- Configure Quality Gate to fail pipeline if score is low
- (Optional) Add Slack alert for gate failures

---

## 🛠️ Tech Stack

- Kubernetes (Minikube)
- ArgoCD
- FastAPI (Python)
- PostgreSQL
- React + NGINX
- SonarQube (optional)

---

## 📄 License

MIT License – for demo and educational use.

---
