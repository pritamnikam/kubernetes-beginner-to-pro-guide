# Advanced: Role of Kubernetes Building Blocks

## Control Plane Components

- `kube-apiserver`: entry point for all cluster operations.
- `etcd`: source of truth for cluster state.
- `kube-scheduler`: assigns Pods to nodes.
- `kube-controller-manager`: runs reconciliation controllers.
- `cloud-controller-manager`: integrates cloud resources.

## Node Components

- `kubelet`: node agent ensuring Pod spec is realized.
- `container runtime`: pulls images and runs containers.
- `kube-proxy`: service networking rules on node.

## Tooling Component

- `kubectl`: API client for users and automation.

## Request Path vs Control Path

- Control path: desired state updates and reconciliation.
- Data path: real app traffic between client and container.

## Command Map

```bash
kubectl get componentstatuses
kubectl get nodes -o wide
kubectl get pods -n kube-system
kubectl api-resources
```

Reference hands-on: [Exercise 11](../exercises/11-kubernetes-building-blocks/README.md)
