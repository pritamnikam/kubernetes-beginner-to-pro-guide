# Exercise 01 — First Deployment + Service + Troubleshooting

## Objective

Deploy a web app with 2 replicas, expose it with a ClusterIP Service, verify connectivity, then troubleshoot a broken deployment.

## What You Will Practice

- Namespace isolation
- Deployment + Service fundamentals
- Selector/label matching
- Basic troubleshooting workflow

## Acceptance Criteria

- Deployment `web` is healthy with 2 ready replicas.
- Service `web` has active endpoints.
- You can access the app locally via port-forward.
- You can identify and fix a deliberately broken manifest.

## Step 0: Prepare Namespace

```bash
kubectl create namespace k8s-lab
kubectl config set-context --current --namespace=k8s-lab
kubectl get ns
```

## Step 1: Deploy Working App

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get all
kubectl rollout status deploy/web
kubectl get endpoints web
```

## Step 2: Verify Access

```bash
kubectl port-forward svc/web 8080:80
```

Open `http://localhost:8080` in your browser. You should see echoserver response details.

## Step 3: Troubleshooting Drill (Broken Manifest)

Apply the broken deployment:

```bash
kubectl apply -f k8s/deployment-broken.yaml
kubectl get pods
kubectl describe pod -l app=web-broken
kubectl get events --sort-by=.lastTimestamp
```

### Your Task

1. Find root cause.
2. Fix the manifest.
3. Re-apply and verify recovery.

Hint: the issue is image tag-related and should appear in Pod events.

## Step 4: Cleanup (Optional)

```bash
kubectl delete -f k8s/
kubectl config set-context --current --namespace=default
kubectl delete namespace k8s-lab
```

## Common Mistakes

- Label mismatch between Service selector and Pod labels.
- Forgetting namespace context.
- Checking only logs but not events.
