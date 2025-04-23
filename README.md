
# ⚙️ GitOps Demo with Argo CD on AKS – TalentLand Edition

This guide walks you through setting up a simple GitOps demo using **Argo CD** and **Azure Kubernetes Service (AKS)**, where application deployments are automated via Git commits.

---

## 🎯 What You’ll Learn

- Create an AKS cluster with a low-cost VM
- Install Argo CD
- Deploy a sample NGINX application using GitOps
- Watch changes in Git automatically sync to the cluster

---

## 🧰 Prerequisites

- Azure CLI (`az`)
- `kubectl` installed and configured
- Git installed
- PowerShell (for Windows users)
- GitHub account

---

## 🚀 1. Deploy AKS with Burstable VM (B2s)

```powershell
$resourceGroup = "talentland-rg"
$aksName = "talentland-aks"
$location = "eastus"

az group create --name $resourceGroup --location $location
az aks create `
  --resource-group $resourceGroup `
  --name $aksName `
  --node-count 1 `
  --node-vm-size Standard_B2s `
  --generate-ssh-keys
az aks get-credentials --resource-group $resourceGroup --name $aksName
```

---

## 📦 2. Install Argo CD

```powershell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## 🌐 3. Access Argo CD UI via Port Forward

```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access the UI at: `https://localhost:8080`

Retrieve the initial admin password:

```powershell
$secret = kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}"
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($secret))
```

---

## 📁 4. Prepare Your Git Repo

Create a folder structure like:

```
.
└── nginx/
    ├── deployment.yaml
    └── service.yaml
```

Example content for `deployment.yaml` (NGINX):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## 🧠 5. Create the Argo CD Application

Create a file named `nginx-app.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/<your-user>/<your-repo>.git
    targetRevision: HEAD
    path: nginx
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply the configuration:

```powershell
kubectl apply -f .\nginx-app.yaml -n argocd
```

---

## 🔄 6. Test GitOps Flow

1. Edit `deployment.yaml` (e.g., change `replicas: 1` → `replicas: 2`)
2. Commit and push the change
3. Argo CD will automatically detect and sync it
4. Check in the cluster:

```powershell
kubectl get pods
```

---

## ✅ Cleanup After the Demo

```powershell
az group delete --name $resourceGroup --yes --no-wait
```

---

## 💡 Tips

- Secure Argo CD with TLS and SSO in real environments
- Try this with Helm charts or Kustomize
- Add Observability (Prometheus, Grafana) for next-level GitOps

---

Inspired by the DevOps & Cloud Practice @ TalentLand 🛠️
