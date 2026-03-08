# Week 8 — Production Delivery with Helm and GitOps

This week maps to Exercise 08 and teaches release standardization.

## Core Concepts

- Helm templates define reusable release units.
- Environment values separate staging and production behavior.
- GitOps overlays represent desired state in Git.
- Rollback rehearsal is a required production skill.

## Why This Matters

Production maturity is consistency, not one-time success.

## Practice Flow

1. Lint and render Helm chart.
2. Validate rendered manifests with client dry-run.
3. Install staging and production releases.
4. Build and validate GitOps overlays.
5. Trigger bad image release and recover with rollback.

Reference lab: [Exercise 08](../exercises/08-production/README.md)

## Essential Commands

```bash
helm lint helm/k8s-app
helm template app-staging helm/k8s-app -f helm/k8s-app/values-staging.yaml > out-staging.yaml
kubectl apply --dry-run=client -f out-staging.yaml
kubectl kustomize gitops/overlays/staging > gitops-staging.yaml
helm rollback app-staging 1 -n k8s-lab
```

## CI/CD Notes

- Validate chart and overlays on every PR.
- Block merges if lint, render, or dry-run fails.
- Keep promotion history in Git commits and release history.

## Cleanup Notes

For PowerShell, use:

```powershell
Remove-Item out-staging.yaml, out-prod.yaml, gitops-staging.yaml, gitops-prod.yaml -ErrorAction SilentlyContinue
```

For bash, use:

```bash
rm -f out-staging.yaml out-prod.yaml gitops-staging.yaml gitops-prod.yaml
```

## Mastery Signals

- You can explain chart values drift between staging and prod.
- You can recover bad release by rollback without manual YAML edits.
- You can describe how GitOps improves auditability and control.
