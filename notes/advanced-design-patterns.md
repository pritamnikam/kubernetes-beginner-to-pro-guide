# Advanced: Common Kubernetes Design Patterns

## Why Patterns Matter

Patterns let you solve recurring architecture problems consistently.

## Common Patterns

- Sidecar: companion container for logs, proxy, or security.
- Init container: one-time setup before app starts.
- Ambassador: local proxy for external dependency.
- Adapter: transform output format for standardized observability.
- Leader election + workers: one coordinator, many executors.
- Operator pattern: controller that extends Kubernetes API.

## Pattern Selection Guide

- Need startup dependency checks: init container.
- Need shared runtime concern: sidecar.
- Need protocol translation: ambassador/adapter.
- Need domain automation: operator.

## Risks and Trade-Offs

- Sidecars add resource overhead and lifecycle coupling.
- Init containers can hide startup bottlenecks.
- Overusing patterns can make simple apps too complex.

## Commands to Observe Patterns

```bash
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod> -c <container-name>
```

Reference hands-on: [Exercise 09](../exercises/09-design-patterns/README.md)
