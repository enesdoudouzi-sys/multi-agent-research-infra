# multi-agent-research-infra

Kubernetes manifests for the [Multi-Agent Research System](https://github.com/enesdoudouzi-sys/multi-agent-research), managed via GitOps with ArgoCD.

## Structure

```
├── base/                        # Base Kubernetes resources
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── ingress.yaml
│   ├── namespace.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/                     # 1 replica, dev image tags, dev hostname
│   └── prod/                    # 3 replicas, pinned image SHAs, prod hostname
└── argocd/
    └── application.yaml         # ArgoCD Application pointing at overlays/prod
```

## GitOps Flow

```
Code push → CI builds image → CI updates image SHA in overlays/prod/ → ArgoCD detects change → deploys to cluster
```

The `overlays/prod/kustomization.yaml` image tags are auto-updated by the CI pipeline in the app repo. **Do not edit them manually.**

## Bootstrap ArgoCD

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Apply the application
kubectl apply -f argocd/application.yaml

# Create the API key secret (once)
kubectl create secret generic api-secrets \
  --from-literal=anthropic-api-key=sk-ant-... \
  -n multi-agent-research
```

## Manual deploy (without ArgoCD)

```bash
# Dev
kubectl apply -k overlays/dev/

# Prod
kubectl apply -k overlays/prod/
```

## Required GitHub Secrets (in app repo)

| Secret | Description |
|--------|-------------|
| `ANTHROPIC_API_KEY` | Used in CI tests |
| `INFRA_REPO_TOKEN` | PAT with write access to this repo (for image tag updates) |
