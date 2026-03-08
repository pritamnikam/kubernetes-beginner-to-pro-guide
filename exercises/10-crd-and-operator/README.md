# Exercise 10 — CRD and Operator Model

## Objective

Learn CRD mechanics and understand where operators fit.

## Step 1: Register CRD

```bash
kubectl apply -f k8s/crd.yaml
kubectl get crd widgets.training.k8s.io
```

## Step 2: Create Custom Resource

```bash
kubectl apply -f k8s/widget-sample.yaml
kubectl get widgets
kubectl get widget sample-widget -o yaml
```

## Step 3: Observe Reality

Without an operator controller, creating a Widget does not create Deployments/Services automatically.

```bash
kubectl get deploy,svc | findstr /I widget
```

## Concept Drill

Explain in your own words:

1. What the API server does for CRD resources.
2. What only an operator can do.
3. Why reconciliation loop is required.

## Cleanup

```bash
kubectl delete -f k8s/widget-sample.yaml
kubectl delete -f k8s/crd.yaml
```
