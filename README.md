# holiday-dashboard-deployment

GitOps deployment for **SOMMER-HQ** — Kustomize manifests reconciled by ArgoCD.
Single environment (**prod**); a commit to `main` deploys directly (no preprod).

## How a deploy happens

1. You push to `main` on the **backend** or **frontend** repo.
2. That repo's CI builds a container image and pushes it to `ghcr.io`.
3. CI clones this repo, runs `kustomize edit set image …:<tag>` in `overlays/prod`,
   and commits the bump back to `main`.
4. ArgoCD (watching `main` / `overlays/prod`, `syncPolicy.automated`) sees the commit
   and reconciles the cluster. No manual `kubectl apply` in the normal flow.

## Layout

```
base/
  backend/    Deployment (8080 http, 8081 mgmt, envFrom backend-secrets) + Service
  frontend/   Deployment (nginx :80) + Service
  postgres/   StatefulSet (postgres:16, 5Gi PVC) + Service
overlays/prod/
  kustomization.yaml   namespace holidash-prod, base + ingress, image tags
  ingress.yaml         nginx + cert-manager, holiday.jannekeipert.de, /api→backend /→frontend
argocd/prod-app.yaml   the ArgoCD Application (auto-sync, prune, self-heal)
docs/secrets-required.md
```

## First-time bootstrap

1. Install ArgoCD and point it at this repo.
2. Create the secrets from [`docs/secrets-required.md`](docs/secrets-required.md).
3. `kubectl apply -f argocd/prod-app.yaml` — ArgoCD creates the `holidash-prod`
   namespace and deploys everything.
