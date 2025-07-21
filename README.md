
# Style Store – Kubernetes + ArgoCD Microservices Demo

This is a demo project showcasing a microservices-based architecture for a fictional e-commerce "Style Store" deployed to Kubernetes with GitOps via ArgoCD.

## Architecture Overview

- **Frontend** (React or Nginx SPA)
- **Auth Service** (FastAPI, PostgreSQL)
- **Orders Service** (FastAPI, PostgreSQL)
- **PostgreSQL** (single shared DB pod with two logical DBs)
- **Ingress** (NGINX Ingress Controller)
- **ArgoCD** (for GitOps-based deployment and sync)
- **SonarQube** (optional, for code quality scanning)

## Project Structure

```
style-store-gitops/
├── apps/
│   ├── auth/
│   ├── orders/
│   ├── frontend/
│   └── init-db/
├── base/
│   ├── postgres/
│   ├── ingress/
│   └── sonarqube/
├── overlays/
│   └── dev/
└── README.md
```

## Ingress Routing

| Path               | Service         | Port |
|--------------------|------------------|------|
| `/api/auth/**`     | `auth-service`   | 8000 |
| `/api/orders/**`   | `orders-service` | 8001 |
| `/` (root)         | `frontend`       | 80   |

Make sure `minikube.local` is added to your `/etc/hosts`:
```bash
echo "$(minikube ip) minikube.local" | sudo tee -a /etc/hosts
```

## Getting Started

### 1. Clone the GitOps Repo
```bash
git clone https://github.com/de2pakchauhan/style-store-gitops.git
cd style-store-gitops
```

### 2. Apply Ingress Controller (NGINX)
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml
```

Wait for the ingress controller pods to be ready:
```bash
kubectl get pods -n ingress-nginx
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

Login:
```bash
argocd login localhost:8080
```

### 4. Setup ArgoCD App

You can create an `Application` resource pointing to your GitHub repo like:
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
    syncOptions:
    - CreateNamespace=true
```

Apply:
```bash
kubectl apply -f app-of-apps.yaml
```

### 5. Initialize Databases (Auth & Orders)

```bash
kubectl apply -f apps/init-db/init-job.yaml
```

It creates `auth_db` and `orders_db` in the PostgreSQL pod.

### 6. Verify

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
```

Open your browser:
```
http://minikube.local/
```

## Optional: SonarQube + Quality Gate CI

Configure SonarQube in a pod with ingress:
- Add your GitHub repo as a project
- Scan using Sonar Scanner in CI
- Fail pipeline if Quality Gate fails
- Optional: Add Slack notifications

---

## License

MIT – for demo/educational purposes.
