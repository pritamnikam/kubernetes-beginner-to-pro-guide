# Advanced: How Kubernetes Works Behind the Scenes

## Reconciliation Loop

Every controller follows:

1. Observe current state.
2. Compare with desired state.
3. Act to reduce drift.
4. Requeue and repeat.

## Scheduling Pipeline (Simplified)

1. Pending Pod enters scheduler queue.
2. Filter nodes by constraints (resources, taints, affinity).
3. Score remaining nodes.
4. Bind Pod to best node.

## Service Routing Internals

- Service selector builds EndpointSlices.
- kube-proxy updates node routing rules.
- Traffic is distributed to healthy endpoints.

## Pod Startup Internals

1. API server accepts Pod spec.
2. Scheduler binds Pod to node.
3. kubelet asks runtime to pull image.
4. Containers start and probes begin.
5. Readiness decides traffic eligibility.

## Useful Debug Sequence

```bash
kubectl get pod <pod> -o yaml
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl get events --sort-by=.lastTimestamp
```

Reference hands-on: [Exercise 12](../exercises/12-behind-the-scenes/README.md)
