# Exercise 03 — ConfigMap, Secret, and Persistent Volume Claim

## Concept First

Phase 3 is where Kubernetes becomes practical for real apps.

- **ConfigMap** stores non-sensitive configuration.
- **Secret** stores sensitive values (credentials, API keys).
- **PVC (PersistentVolumeClaim)** gives Pods durable storage independent of Pod lifecycle.

Core idea: container images stay generic, while runtime config/state is injected by Kubernetes.

## Objective

1. Inject runtime config from a ConfigMap.
2. Inject sensitive data from a Secret.
3. Persist application data to a PVC.
4. Troubleshoot a configuration failure (`CreateContainerConfigError`).

## Acceptance Criteria

- `ConfigMap`, `Secret`, and `PVC` are created successfully.
- Deployment `stateful-web` is `Available`.
- PVC `app-data-pvc` is `Bound`.
- After deleting the Pod, data in `/data` still exists in the recreated Pod.
- You diagnose and fix the broken deployment drill.

## Prerequisites

- Active namespace context: `k8s-lab`
- Working cluster and `kubectl`

## Step 1: Apply Config and Storage Resources

```bash
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/pvc.yaml
kubectl get configmap,secret,pvc
```

Expected: `app-data-pvc` should become `Bound`.

## Step 2: Deploy App Using Config + Secret + PVC

```bash
kubectl apply -f k8s/deployment.yaml
kubectl rollout status deploy/stateful-web
kubectl get pods -l app=stateful-web -o wide
kubectl logs -l app=stateful-web --tail=30
```

What to look for in logs:

- Config file content loaded from `/etc/app-config/app.properties`
- Confirmation that secret key is present (without printing secret value)

## Step 3: Verify Data Persistence

Get Pod name:

```bash
kubectl get pods -l app=stateful-web
```

Write custom data:

```bash
kubectl exec deploy/stateful-web -- sh -c "echo learning-k8s >> /data/custom.log; tail -n 5 /data/custom.log"
```

Delete the Pod and let Deployment recreate it:

```bash
kubectl delete pod -l app=stateful-web
kubectl rollout status deploy/stateful-web
kubectl exec deploy/stateful-web -- sh -c "tail -n 5 /data/custom.log"
```

Expected: previously written `learning-k8s` entry still exists.

## Step 4: Break/Fix Drill (Config Error)

Apply broken deployment:

```bash
kubectl apply -f k8s/deployment-broken.yaml
kubectl get pods -l app=stateful-web-broken
kubectl describe pod -l app=stateful-web-broken
```

Task:

1. Identify why Pod is not starting.
2. Fix `secretKeyRef` in the broken manifest.
3. Re-apply and verify Pod starts.

## Cleanup (Optional)

```bash
kubectl delete -f k8s/deployment-broken.yaml
kubectl delete -f k8s/deployment.yaml
kubectl delete -f k8s/pvc.yaml
kubectl delete -f k8s/secret.yaml
kubectl delete -f k8s/configmap.yaml
```

## Common Mistakes

- Putting secrets in ConfigMap instead of Secret.
- Assuming data is persistent without using PVC.
- Forgetting to inspect `describe` output for `CreateContainerConfigError` details.
