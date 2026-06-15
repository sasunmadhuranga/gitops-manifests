# gitops-manifests

Kubernetes manifests for the GitOps + ArgoCD on EKS project.
This is the **source of truth** for what is deployed to the cluster.
No one deploys manually — all changes go through Git.

## Structure

```
gitops-manifests/
├── argocd/
│   └── application.yaml          ← Apply once to register the app with ArgoCD
└── manifests/
    └── app/
        ├── base/                 ← Shared manifests (Deployment, Service, Ingress, Namespace)
        │   ├── kustomization.yaml
        │   ├── namespace.yaml
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   └── ingress.yaml
        └── overlays/
            ├── dev/              ← Dev environment (ArgoCD watches this)
            │   └── kustomization.yaml   ← GitHub Actions updates newTag here
            └── prod/             ← Prod environment (manual PR to promote)
                └── kustomization.yaml
```

## One-time setup

### 1. Replace placeholder values

In `manifests/app/overlays/dev/kustomization.yaml` and `overlays/prod/kustomization.yaml`:
- Replace `REPLACE_WITH_YOUR_ECR_URL` with the actual ECR URL from `terraform output ecr_repository_url`

In `argocd/application.yaml`:
- Replace `YOUR_GITHUB_USERNAME` with your GitHub username

### 2. Register the app with ArgoCD

```bash
# Make sure kubectl is pointing at your EKS cluster
aws eks update-kubeconfig --region us-east-1 --name gitops-argocd-dev-cluster

# Apply the ArgoCD Application resource
kubectl apply -f argocd/application.yaml

# Watch ArgoCD sync in the UI (port-forward if not already open)
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Open http://localhost:8080 → app should appear and start syncing
```

### 3. Set up GitHub secrets in your app repo

Go to your **python-backend-app** repo → Settings → Secrets and variables → Actions:

| Secret | Value |
|---|---|
| `AWS_ACCESS_KEY_ID` | IAM user access key with ECR push permissions |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key |
| `GITOPS_PAT` | GitHub PAT with `Contents: write` on this repo |

To create a fine-grained PAT:
GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → New token
- Repository access: Only select `gitops-manifests`
- Permissions: Contents → Read and write

## How a deployment works

```
1. You push to master in python-backend-app
2. GitHub Actions builds the Docker image and pushes to ECR with tag sha-<commit>
3. GitHub Actions checks out THIS repo and updates newTag in overlays/dev/kustomization.yaml
4. GitHub Actions commits and pushes that single-line change
5. ArgoCD detects the new commit (polls every 3 min by default)
6. ArgoCD applies the updated manifests to the EKS cluster
7. Kubernetes performs a rolling update — zero downtime
```

## Rolling back a bad deploy

```bash
# Option 1: Revert the last commit in this repo
git revert HEAD
git push
# ArgoCD detects the revert and rolls the cluster back automatically

# Option 2: Manually set a specific tag
# Edit overlays/dev/kustomization.yaml → change newTag to a previous sha-xxxxx
git commit -am "rollback: revert to sha-abc1234"
git push
```
