# Exercise 02 — Rollout, History, and Rollback

## Concept First

A `Deployment` creates a new revision when its Pod template changes (image, env vars, command, labels in template, etc.).

- **Rollout**: moving from old ReplicaSet to new ReplicaSet gradually.
- **History**: stored revisions of Deployment changes.
- **Rollback**: return to previous known-good revision.

This is a core production skill: deploy safely, observe, and recover quickly.

## Objective

1. Deploy baseline `web` revision (`v1`).
2. Perform a safe rollout to `v2`.
3. Trigger a broken rollout.
4. Inspect rollout history and rollback to stable revision.

## Acceptance Criteria

- You can show deployment revision history.
- You can identify a failed rollout from status/events.
- You can rollback and restore all pods to healthy state.

## Prerequisites

- Active namespace context: `k8s-lab`
- Working cluster and `kubectl`

## Step 1: Apply Baseline (v1)

```bash
kubectl apply -f k8s/deployment-v1.yaml
kubectl rollout status deploy/web
kubectl get deploy web
kubectl get rs -l app=web
```

## Step 2: Safe Rollout (v2)

This rollout changes environment variable `RELEASE` from `v1` to `v2` (same image).

```bash
kubectl apply -f k8s/deployment-v2.yaml
kubectl rollout status deploy/web
kubectl rollout history deploy/web
kubectl get rs -l app=web
```

## Step 3: Broken Rollout Drill

This rollout introduces an invalid image tag.

```bash
kubectl apply -f k8s/deployment-broken.yaml
kubectl rollout status deploy/web --timeout=60s
kubectl get pods -l app=web
kubectl describe pod -l app=web
kubectl rollout history deploy/web
```

## Step 4: Recover with Rollback

```bash
kubectl rollout undo deploy/web
kubectl rollout status deploy/web
kubectl get pods -l app=web -o wide
kubectl rollout history deploy/web
```

## Optional: Rollback to Specific Revision

```bash
kubectl rollout undo deploy/web --to-revision=2
```

## Cleanup (Optional)

```bash
kubectl delete -f k8s/
```

## Common Mistakes

- Using `kubectl get pods` only; missing rollout insights from `kubectl rollout`.
- Not checking ReplicaSets during rollout.
- Forgetting that failed image pull can block revision progress.
