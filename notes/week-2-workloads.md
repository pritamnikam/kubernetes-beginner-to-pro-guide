# Week 2 — Workloads, Rollouts, and Rollbacks

This week maps to Exercise 02 and teaches controlled change in Kubernetes.

## Core Concepts

- A `Deployment` creates a new ReplicaSet whenever the Pod template changes.
- Rollout is gradual replacement of old Pods with new Pods.
- Rollback restores a previous known-good revision.
- ReplicaSet history is your release timeline.

## Why This Matters

In production, the difference between a safe release and an outage is usually rollout discipline.

## Practice Flow

1. Apply baseline deployment (`v1`).
2. Apply safe update (`v2`).
3. Inspect revision history and ReplicaSets.
4. Apply broken rollout and observe failure.
5. Recover quickly with `kubectl rollout undo`.

Reference lab: [Exercise 02](../exercises/02-rollout-and-rollback/README.md)

## Essential Commands

```bash
kubectl rollout status deploy/web
kubectl rollout history deploy/web
kubectl get rs -l app=web
kubectl rollout undo deploy/web
kubectl rollout undo deploy/web --to-revision=2
```

## Debug Checklist

1. Check rollout progress and timeout.
2. Check new ReplicaSet Pod status.
3. Inspect events for image pull errors.
4. Roll back first, then investigate deeper.

## Mastery Signals

- You can explain which field changes create a new revision.
- You can identify the bad revision from history.
- You can restore service in under 2 minutes using rollback.
