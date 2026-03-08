# Exercise 08 — Helm, CI/CD Validation, and GitOps Readiness

## Concept First

Phase 8 is about repeatable production delivery.

- **Helm**: package Kubernetes resources as reusable, versioned templates.
- **CI/CD Validation**: fail fast by linting and dry-running manifests before deploy.
- **GitOps**: store desired state in Git and promote environments through PRs.

Core idea: production quality comes from predictable release mechanics.

## Objective

1. Package an app with Helm chart templates.
2. Validate chart rendering and Kubernetes schema checks.
3. Deploy with environment-specific values (staging/prod).
4. Prepare GitOps-style base + overlays with Kustomize.
5. Practice break/fix by introducing invalid image tag and recovering via rollback.

## Acceptance Criteria

- Helm chart installs successfully in `k8s-lab`.
- Staging and production values render different replica/image settings.
- GitOps overlays build cleanly with `kubectl kustomize`.
- CI validation commands complete (`helm lint`, `helm template`, `kubectl apply --dry-run=client`).
- Rollback restores workload after bad release.

## Prerequisites

- Active namespace context: `k8s-lab`
- `kubectl` installed
- `helm` installed

## Step 1: Validate Helm Chart

```bash
helm lint helm/k8s-app
helm template app-staging helm/k8s-app -f helm/k8s-app/values-staging.yaml > out-staging.yaml
helm template app-prod helm/k8s-app -f helm/k8s-app/values-prod.yaml > out-prod.yaml
kubectl apply --dry-run=client -f out-staging.yaml
kubectl apply --dry-run=client -f out-prod.yaml
```

## Step 2: Install Staging Release

```bash
helm upgrade --install app-staging helm/k8s-app -f helm/k8s-app/values-staging.yaml -n k8s-lab
kubectl get deploy,svc -l app.kubernetes.io/instance=app-staging -n k8s-lab
kubectl rollout status deploy/app-staging-k8s-app -n k8s-lab
```

## Step 3: Promote to Production Values

```bash
helm upgrade --install app-prod helm/k8s-app -f helm/k8s-app/values-prod.yaml -n k8s-lab
kubectl get deploy,svc -l app.kubernetes.io/instance=app-prod -n k8s-lab
kubectl rollout status deploy/app-prod-k8s-app -n k8s-lab
```

## Step 4: GitOps Overlay Checks

```bash
kubectl kustomize gitops/overlays/staging > gitops-staging.yaml
kubectl kustomize gitops/overlays/prod > gitops-prod.yaml
kubectl apply --dry-run=client -f gitops-staging.yaml
kubectl apply --dry-run=client -f gitops-prod.yaml
```

## Step 5: Break/Fix Drill (Bad Image Tag + Rollback)

Break release:

```bash
helm upgrade app-staging helm/k8s-app -f helm/k8s-app/values-staging.yaml --set image.tag=does-not-exist -n k8s-lab
kubectl rollout status deploy/app-staging-k8s-app -n k8s-lab --timeout=60s
kubectl get pods -l app.kubernetes.io/instance=app-staging -n k8s-lab
```

Fix with rollback:

```bash
helm history app-staging -n k8s-lab
helm rollback app-staging 1 -n k8s-lab
kubectl rollout status deploy/app-staging-k8s-app -n k8s-lab
```

## Step 6: CI Example

A ready-to-copy CI workflow exists at `ci/github-actions-example.yaml`.
It validates chart lint, render, and client-side apply checks.

## Cleanup (Optional)

```bash
helm uninstall app-staging -n k8s-lab
helm uninstall app-prod -n k8s-lab
rm out-staging.yaml out-prod.yaml gitops-staging.yaml gitops-prod.yaml
```

## Common Mistakes

- Deploying directly without pre-validation.
- Mixing staging and prod values in the same release.
- Skipping rollback rehearsal.
- Treating GitOps as just a folder structure instead of immutable desired state.
