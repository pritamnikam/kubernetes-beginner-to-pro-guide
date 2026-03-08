# Exercise 09 — Advanced Design Patterns

## Objective

Practice three common Kubernetes design patterns:

1. Init container
2. Sidecar container
3. Worker job pattern

## Prerequisites

- Namespace context set to `k8s-lab`

## Step 1: Init + Sidecar Deployment

```bash
kubectl apply -f k8s/deployment-init-sidecar.yaml
kubectl get pods -l app=pattern-app
kubectl describe pod -l app=pattern-app
kubectl logs -l app=pattern-app -c main --tail=30
kubectl logs -l app=pattern-app -c sidecar --tail=30
```

Observe:

- Init container runs first and exits successfully.
- Main and sidecar run together after init completion.

## Step 2: Worker Job Pattern

```bash
kubectl apply -f k8s/job-worker.yaml
kubectl get jobs
kubectl logs job/pattern-worker-job
```

Observe one-shot execution and completion status.

## Break/Fix Drill

Edit sidecar command to an invalid shell command and re-apply. Then diagnose with:

```bash
kubectl describe pod -l app=pattern-app
kubectl logs -l app=pattern-app -c sidecar --previous
```

## Cleanup

```bash
kubectl delete -f k8s/
```
