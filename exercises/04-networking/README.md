# Exercise 04 — DNS, Ingress, and NetworkPolicy

## Concept First

Phase 4 focuses on how traffic flows inside and into a cluster.

- **Service DNS** gives stable names like `net-web` and `net-web.k8s-lab.svc.cluster.local`.
- **Ingress** provides HTTP routing from outside the cluster to Services.
- **NetworkPolicy** controls which Pods can talk to which Pods.

Core idea: networking in Kubernetes is declarative and label-driven.

## Objective

1. Deploy a simple web app and expose it with a Service.
2. Validate in-cluster DNS/service discovery from client Pods.
3. Enforce traffic policy with `NetworkPolicy` (deny by default, allow specific client).
4. Configure Ingress route (if ingress controller exists).
5. Troubleshoot a broken Service selector.

## Acceptance Criteria

- Deployment `net-web` is healthy.
- Service `net-web` has endpoints.
- Allowed client Pod can reach Service, denied client cannot.
- Ingress object is created; traffic works if an ingress controller is installed.
- Broken Service drill is diagnosed and fixed.

## Prerequisites

- Active namespace context: `k8s-lab`
- Working cluster and `kubectl`

## Step 1: Deploy App + Service + Client Pods

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/client-allowed.yaml
kubectl apply -f k8s/client-denied.yaml
kubectl rollout status deploy/net-web
kubectl get pods,svc -l app=net-web
kubectl get endpoints net-web
```

## Step 2: Test DNS and Service Access (before NetworkPolicy)

```bash
kubectl exec net-client-allowed -- wget -qO- http://net-web
kubectl exec net-client-allowed -- wget -qO- http://net-web.k8s-lab.svc.cluster.local
kubectl exec net-client-denied -- wget -qO- http://net-web
```

Expected: both clients can access before policy is applied.

## Step 3: Apply Network Policies

```bash
kubectl apply -f k8s/networkpolicy-deny-all.yaml
kubectl apply -f k8s/networkpolicy-allow-from-allowed-client.yaml
kubectl get networkpolicy
```

Retest:

```bash
kubectl exec net-client-allowed -- wget -T 2 -qO- http://net-web
kubectl exec net-client-denied -- wget -T 2 -qO- http://net-web
```

Expected:

- `net-client-allowed` succeeds.
- `net-client-denied` times out or fails.

## Step 4: Ingress (Optional if controller installed)

Apply ingress object:

```bash
kubectl apply -f k8s/ingress.yaml
kubectl get ingress
```

Check if controller exists:

```bash
kubectl get pods -A | findstr /I ingress
```

If an ingress controller is available, add host entry:

```text
127.0.0.1 net-web.local
```

Then access:

```bash
kubectl port-forward svc/net-web 8080:80
```

Open `http://net-web.local:8080`.

## Step 5: Break/Fix Drill (Service Selector)

Apply broken service:

```bash
kubectl apply -f k8s/service-broken.yaml
kubectl get endpoints net-web-broken
```

Task:

1. Find why endpoints are empty.
2. Fix selector to match deployment label `app: net-web`.
3. Re-apply and verify endpoints are populated.

## Cleanup (Optional)

```bash
kubectl delete -f k8s/
```

## Common Mistakes

- Confusing Service port with container port.
- Assuming NetworkPolicy works without checking labels/selectors.
- Expecting Ingress routing without an installed ingress controller.
