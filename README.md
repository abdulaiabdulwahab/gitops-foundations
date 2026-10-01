# gitops-foundations

# GitOps Foundations with Flux and AKS

This project demonstrates the fundamentals of **GitOps** using **Flux v2**, **GitHub**, and **Azure Kubernetes Service (AKS)**.

The goal is to use Git as the **single source of truth** for Kubernetes configuration and allow Flux to automatically reconcile the AKS cluster with the configuration stored in the repository.

## Project Architecture

```text
Developer
   |
   v
GitHub Repository
   |
   v
Flux
   |
   v
Azure Kubernetes Service
   |
   v
Nginx Application
```

## Technologies Used

- Azure Kubernetes Service (AKS)
- Flux v2
- GitHub
- Kubernetes
- Kustomize
- Azure CLI
- kubectl
- Git

## Project Structure

```text
gitops-foundations/
|
└── clusters/
    └── dev/
        ├── namespace.yaml
        ├── deployment.yaml
        ├── service.yaml
        └── kustomization.yaml
```

## What This Project Demonstrates

This project covers the following GitOps concepts:

- Git as the source of truth
- Declarative Kubernetes configuration
- Flux reconciliation
- Automatic Kubernetes deployments from Git
- Configuration drift detection
- Automatic drift correction
- Kustomize
- Resource pruning

## Deployment Process

### 1. Create the Azure Resource Group

```bash
az group create \
  --name rg-gitops-foundations \
  --location canadacentral
```

### 2. Create the AKS Cluster

```bash
az aks create \
  --resource-group rg-gitops-foundations \
  --name aks-gitops-foundations \
  --location canadacentral \
  --node-count 1 \
  --enable-managed-identity \
  --generate-ssh-keys
```

### 3. Connect kubectl to AKS

```bash
az aks get-credentials \
  --resource-group rg-gitops-foundations \
  --name aks-gitops-foundations \
  --overwrite-existing
```

Verify the cluster:

```bash
kubectl get nodes
```

### 4. Push the Kubernetes Configuration to GitHub

```bash
git add .

git commit -m "Add initial GitOps configuration"

git push
```

### 5. Configure Flux

Flux monitors the GitHub repository and automatically deploys the Kubernetes manifests stored in:

```text
clusters/dev
```

Example Flux configuration:

```bash
az k8s-configuration flux create \
  --resource-group "$RG" \
  --cluster-name "$AKS" \
  --cluster-type managedClusters \
  --name gitops-foundations \
  --namespace flux-system \
  --scope cluster \
  --url "$GIT_REPO" \
  --branch main \
  --kustomization \
    name=apps \
    path=./clusters/dev \
    prune=true \
    sync_interval=1m \
    retry_interval=1m
```

## Verify Flux

Check the Flux controllers:

```bash
kubectl get pods -n flux-system
```

Check the Git source and Kustomizations:

```bash
kubectl get gitrepositories,kustomizations -A
```

Verify the application:

```bash
kubectl get all -n gitops-demo
```

## Testing GitOps Reconciliation

The application initially runs with:

```yaml
replicas: 2
```

Change this value in Git to:

```yaml
replicas: 4
```

Commit and push the change:

```bash
git add .

git commit -m "Scale application to four replicas"

git push
```

Flux detects the change and automatically updates AKS.

Verify:

```bash
kubectl get deployment hello-gitops \
  -n gitops-demo
```

## Testing Configuration Drift

Manually change the running deployment:

```bash
kubectl scale deployment hello-gitops \
  --namespace gitops-demo \
  --replicas=1
```

Git still defines:

```yaml
replicas: 4
```

Flux detects the configuration drift and automatically restores the deployment to four replicas.

This demonstrates one of the main benefits of GitOps:

```text
Git Desired State
       |
       v
Flux Reconciliation
       |
       v
AKS Actual State
```

## Troubleshooting Commands

Check Flux resources:

```bash
kubectl get gitrepositories -A

kubectl get kustomizations -A
```

Check the Flux source controller:

```bash
kubectl logs \
  -n flux-system \
  deployment/source-controller
```

Check the Kustomize controller:

```bash
kubectl logs \
  -n flux-system \
  deployment/kustomize-controller
```

Validate Kubernetes manifests locally:

```bash
kubectl apply \
  --dry-run=client \
  -k clusters/dev
```

## Key Lessons

This project demonstrates that with GitOps:

- Git stores the desired application state.
- Flux continuously monitors Git.
- Kubernetes changes are deployed automatically.
- Manual configuration changes can be automatically corrected.
- Git provides an auditable history of infrastructure and application changes.
- Kubernetes deployments can be managed without manually running `kubectl apply`.

## Cleanup

Delete the Azure resource group when the lab is complete:

```bash
az group delete \
  --name rg-gitops-foundations \
  --yes \
  --no-wait
```

## Summary

This project provided hands-on experience with the core GitOps workflow:

```text
Git Change
    |
    v
Commit
    |
    v
Push to GitHub
    |
    v
Flux Detects Change
    |
    v
AKS Reconciles
    |
    v
Desired State Achieved
```

The project forms a foundation for more advanced GitOps implementations involving multiple environments, Helm, ACR, Azure Pipelines, AKS, security controls, and production deployment workflows.