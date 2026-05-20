# exploring_minikube_k8s

## Folder Structure
```text
exploring_minikube_k8s/
│
└── k8s_chart/
    ├── Chart.yaml                  
    ├── values.yaml                
    └── templates/
        ├── namespace.yaml
        ├── configmap.yaml
        ├── redis-deployment.yaml
        ├── redis-service.yaml
        ├── redis-pvc.yaml
        ├── frontend-deployment.yaml
        ├── frontend-service.yaml
        └── ingress.yaml
```
# Guestbook K8s — Minikube on Windows

A simple guestbook app deployed on Kubernetes using Minikube and Helm.
Stack: nginx (frontend) + Redis (database).

---

## Prerequisites

Install the following:
- [Docker Desktop](https://www.docker.com/products/docker-desktop) — must be running in the background
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://github.com/helm/helm/releases) — download `windows-amd64.zip`, extract, move `helm.exe` to `C:\Windows\System32\`

Verify installs:
```powershell
docker --version
minikube version
kubectl version --client
helm version
```

---

## Folder Structure

```
guestbook-chart/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── namespace.yaml
    ├── configmap.yaml
    ├── redis-pvc.yaml
    ├── redis-deployment.yaml
    ├── redis-service.yaml
    ├── frontend-deployment.yaml
    ├── frontend-service.yaml
    └── ingress.yaml
```

---

## Step 1 — Start Minikube

```powershell
minikube start
minikube addons enable ingress
kubectl get nodes
```

Wait until node shows `Ready`.

---

## Step 2 — Validate the Helm Chart

Run from the folder **containing** `guestbook-chart/`:

```powershell
helm lint ./guestbook-chart
helm template guestbook ./guestbook-chart
```

---

## Step 3 — Deploy with Helm

```powershell
helm install guestbook ./guestbook-chart --create-namespace --namespace guestbook
```

Verify everything is running:

```powershell
kubectl get all -n guestbook
kubectl get pvc -n guestbook
kubectl get ingress -n guestbook
```

Wait until all Pods show `Running` and PVC shows `Bound`.

---

## Step 4 — Access the App

### Option A — Port Forward (recommended on Windows)

```powershell
kubectl port-forward service/frontend 8080:80 -n guestbook
```

Open browser and go to:
```
http://localhost:8080
```

### Option B — Via Ingress + minikube tunnel

**Terminal 1** (PowerShell as Administrator, keep open):
```powershell
minikube tunnel
```

**Terminal 2:**
```powershell
kubectl get ingress -n guestbook
```

Check the ADDRESS column. If it shows `127.0.0.1`, open Notepad as Administrator and edit:
```
C:\Windows\System32\drivers\etc\hosts
```

Add this line at the bottom:
```
127.0.0.1 guestbook.local
```

Flush DNS:
```powershell
ipconfig /flushdns
```

Open browser and go to:
```
http://guestbook.local
```

---

## Useful Commands

```powershell
# See all resources
kubectl get all -n guestbook

# Check logs
kubectl logs -n guestbook -l app=frontend

# Shell into a pod
kubectl exec -it -n guestbook deploy/frontend -- /bin/sh

# Scale frontend
kubectl scale deployment frontend -n guestbook --replicas=5

# Watch pods in real time
kubectl get pods -n guestbook -w

# Upgrade after changing values.yaml
helm upgrade guestbook ./guestbook-chart

# Rollback
helm rollback guestbook 1
```

---

## Cleanup

```powershell
helm uninstall guestbook
minikube stop
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Pods stuck in `Pending` | `kubectl describe pod <name> -n guestbook` |
| PVC stuck in `Pending` | `kubectl describe pvc -n guestbook` |
| Ingress ADDRESS empty | Wait 2 mins after `minikube addons enable ingress` |
| `guestbook.local` not reachable | Use port-forward instead, or run `minikube tunnel` as Admin |
| nginx not showing | Check pods are Running: `kubectl get all -n guestbook` |
