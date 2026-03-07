# Exercise 07 — Logs, Metrics, and Troubleshooting

## Concept First

Phase 7 is about fast and repeatable debugging.

- **Logs** tell you what happened inside containers.
- **Describe + Events** tell you what Kubernetes did to your workload.
- **Metrics** (CPU/memory) reveal pressure and scaling signals.
- **Troubleshooting flow** turns incidents into a method, not guesswork.

Core idea: use a consistent triage order and narrow down root cause quickly.

## Objective

1. Deploy an app that emits logs and exposes a Service.
2. Practice log inspection (`logs`, `logs -f`, `logs --previous`).
3. Use `describe` and `events` to diagnose lifecycle issues.
4. Use `top` metrics if metrics-server is available.
5. Fix a crashing deployment from observed evidence.

## Acceptance Criteria

- `obs-web` deployment is healthy and reachable through Service.
- You can stream logs and identify normal heartbeat entries.
- You can identify crash reason in `obs-web-broken` using events + previous logs.
- You can restore healthy state by applying fixed manifest.

## Prerequisites

- Active namespace context: `k8s-lab`
- Working cluster and `kubectl`

## Step 1: Deploy Healthy Observability Target

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl rollout status deploy/obs-web
kubectl get pods -l app=obs-web -o wide
kubectl get svc obs-web
```

## Step 2: Logs Practice

Tail logs:

```bash
kubectl logs -l app=obs-web --tail=50
kubectl logs -f -l app=obs-web
```

You should see heartbeat lines and periodic status messages.

## Step 3: Describe + Events Practice

```bash
kubectl describe pod -l app=obs-web
kubectl get events --sort-by=.lastTimestamp | findstr /I obs-web
```

Goal: correlate pod lifecycle events with app logs.

## Step 4: Metrics Practice (Optional)

```bash
kubectl top pods -l app=obs-web
kubectl top nodes
```

If this fails, metrics-server is not installed or ready.

## Step 5: Break/Fix Drill (CrashLoopBackOff)

Apply broken deployment:

```bash
kubectl apply -f k8s/deployment-broken.yaml
kubectl get pods -l app=obs-web-broken
kubectl describe pod -l app=obs-web-broken
kubectl logs -l app=obs-web-broken --previous --tail=30
kubectl get events --sort-by=.lastTimestamp | findstr /I obs-web-broken
```

Task:

1. Identify crash reason from logs/events.
2. Apply fix manifest.
3. Verify rollout recovers.

```bash
kubectl apply -f k8s/deployment-fixed.yaml
kubectl rollout status deploy/obs-web-broken
kubectl get pods -l app=obs-web-broken
```

## Cleanup (Optional)

```bash
kubectl delete -f k8s/deployment-fixed.yaml --ignore-not-found
kubectl delete -f k8s/deployment-broken.yaml --ignore-not-found
kubectl delete -f k8s/deployment.yaml --ignore-not-found
kubectl delete -f k8s/service.yaml --ignore-not-found
```

## Common Mistakes

- Looking only at pod status without checking logs.
- Forgetting `logs --previous` for restarting containers.
- Ignoring events when root cause is scheduler/kubelet-level.
- Assuming metrics are available in all local clusters.
