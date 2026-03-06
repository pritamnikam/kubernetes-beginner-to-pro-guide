# Exercise 05 — Probes, Resources, and Autoscaling

## Concept First

Phase 5 is about keeping apps healthy under real conditions.

- **Liveness probe**: restarts container when it is stuck/unhealthy.
- **Readiness probe**: controls whether Pod receives traffic.
- **Startup probe**: gives slow-starting apps time before liveness begins.
- **Requests/Limits**: reserve and cap CPU/memory to improve scheduling and cluster stability.
- **HPA**: scales replicas automatically based on metrics (usually CPU).

Core idea: reliability is proactive, not reactive.

## Objective

1. Deploy a web app with startup/readiness/liveness probes.
2. Add resource requests/limits.
3. Configure HPA for CPU-based scaling (if metrics server exists).
4. Run a probe failure break/fix drill.

## Acceptance Criteria

- Deployment `reliable-web` becomes available.
- Pod shows all probes configured and healthy.
- Resource requests/limits are visible in Pod spec.
- HPA object is created; scaling behavior can be observed if metrics are available.
- Broken deployment is diagnosed and fixed.

## Prerequisites

- Active namespace context: `k8s-lab`
- Working cluster and `kubectl`

## Step 1: Deploy Reliable Baseline

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl rollout status deploy/reliable-web
kubectl get pods -l app=reliable-web -o wide
kubectl describe pod -l app=reliable-web
```

Check in `describe` output:

- `startupProbe`, `readinessProbe`, `livenessProbe`
- CPU/memory requests and limits

## Step 2: Create HPA (Optional if metrics-server installed)

```bash
kubectl apply -f k8s/hpa.yaml
kubectl get hpa reliable-web-hpa
kubectl top pods
```

If `kubectl top` fails, install/enable metrics-server first and continue.

## Step 3: Generate Load (for HPA observation)

```bash
kubectl apply -f k8s/load-generator.yaml
kubectl logs load-generator --tail=20
kubectl get hpa reliable-web-hpa -w
```

Expected (with metrics-server): replicas may scale above 2 when CPU crosses threshold.

## Step 4: Break/Fix Drill (Bad Probe Port)

```bash
kubectl apply -f k8s/deployment-broken.yaml
kubectl get pods -l app=reliable-web-broken
kubectl describe pod -l app=reliable-web-broken
```

Task:

1. Identify why pod is repeatedly failing readiness/liveness.
2. Fix probe port to the app container port.
3. Re-apply and confirm pod becomes ready.

## Cleanup (Optional)

```bash
kubectl delete -f k8s/load-generator.yaml --ignore-not-found
kubectl delete -f k8s/hpa.yaml --ignore-not-found
kubectl delete -f k8s/deployment-broken.yaml --ignore-not-found
kubectl delete -f k8s/deployment.yaml --ignore-not-found
kubectl delete -f k8s/service.yaml --ignore-not-found
```

## Common Mistakes

- Using liveness probe where startup probe is needed.
- Setting resource limits too low and causing throttling/OOM.
- Expecting HPA without metrics-server.
- Not checking `describe` events when probes fail.
