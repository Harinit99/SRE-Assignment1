
# Counter App Deployment using KIND, Helm, ArgoCD, and Ingress

## Overview

This project demonstrates the deployment of a containerized Counter application using:
- A local Kubernetes cluster set up using **KIND** (Kubernetes in Docker)
- **Helm** for package management
- **ArgoCD** for GitOps-based continuous deployment
- **NGINX Ingress Controller** for routing
- The app exposes a `/counter` endpoint that increments with every request.

---

## Tools & Technologies Used

- KIND (Kubernetes in Docker)
- Helm
- ArgoCD
- GitHub (as Git repo source)
- NGINX Ingress Controller

---

## Deployment Steps

### 1. Create a KIND Cluster
Created a Kubernetes cluster using KIND:
```
kind create cluster --name dev-cluster
```

### 2. Installed ArgoCD
Installed ArgoCD and port-forwarded the UI:
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Logged in to ArgoCD UI using:
- Username: `admin`
- Password: Retrieved using `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($_)) }`

### 3. Helm Chart Creation
Created a Helm chart named `metrics-app`, which included:
- `deployment.yaml`
- `service.yaml`
- `secret.yaml` (for setting the password as an environment variable)

### 4. GitOps Configuration in ArgoCD UI
- Opened ArgoCD UI and created a new app called `counter-app`.
- Set the Git repo URL to the one containing the Helm chart.
- Chose the path inside the repo where `Chart.yaml` is located.
- Set the destination cluster and namespace.

App synced successfully, but stayed in **“Progressing”** state due to missing Ingress controller.

---

### Installed NGINX Ingress
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/kind/deploy.yaml
```
## Issues Faced & Solutions

### App Stuck in “Progressing” State
- **Cause**: Ingress controller pod was stuck in `Pending` state.
- **Error**: `0/1 nodes are available: node(s) didn't match Pod's node affinity/selector`.
- **Fix**: Realized the ingress controller couldn't be scheduled due to nodeSelector or taint issue.

```
kubectl label node dev-cluster-control-plane ingress-ready=true
kubectl delete pod -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

After successful install, the app health moved from **Progressing → Healthy**.


## Validating the Application

Verified with:
```powershell
curl http://localhost:8080/counter
```

Ran the following curl loop to verify `/counter` increments:
```powershell
1..10 | ForEach-Object { curl http://localhost:8080/counter }
```

Confirmed that the counter value was incrementing as expected.

---

## Final Status

- Counter app successfully deployed and accessible via `localhost/counter`.
- Application is healthy in ArgoCD.
- Helm chart is version-controlled in GitHub.

---
