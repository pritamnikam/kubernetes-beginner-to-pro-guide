# Exercise 12 — Behind the Scenes Workflow

## Objective

Trace what happens internally from `kubectl apply` to running workload.

## Step 1: Apply a Deployment

```bash
kubectl apply -f ../01-first-deployment/k8s/deployment.yaml
kubectl get deploy web -o yaml
```

## Step 2: Watch Scheduling and Startup

```bash
kubectl get pods -l app=web -w
kubectl get events --sort-by=.lastTimestamp
```

## Step 3: Correlate with Control Loops

Answer these with evidence from events/output:

1. When did scheduler bind the pod?
2. When did kubelet pull image and start container?
3. When did deployment controller mark available replicas?

## Step 4: Service Endpoints Internals

```bash
kubectl apply -f ../01-first-deployment/k8s/service.yaml
kubectl get endpoints web -o yaml
kubectl get endpointslices -l kubernetes.io/service-name=web -o yaml
```

Explain how Service selector produced endpoint data consumed by kube-proxy.

## Cleanup

```bash
kubectl delete -f ../01-first-deployment/k8s/
```
