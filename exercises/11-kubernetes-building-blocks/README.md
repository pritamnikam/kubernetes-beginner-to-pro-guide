# Exercise 11 — Kubernetes Building Blocks Deep Dive

## Objective

Map each Kubernetes building block to its real role and observe it in your cluster.

## Step 1: Inspect Core Components

```bash
kubectl get nodes -o wide
kubectl get pods -n kube-system
kubectl get pods -n kube-system -o wide
```

Identify:

- `kube-apiserver`
- `kube-scheduler`
- `kube-controller-manager`
- `etcd`
- `kube-proxy`

## Step 2: Link Components to Responsibilities

For each component, write one sentence:

1. What state it reads.
2. What state it writes.
3. What happens if it fails.

## Step 3: kubectl as API Client

```bash
kubectl api-resources
kubectl get --raw /healthz
```

Explain how `kubectl` talks to API server instead of directly to kubelet or etcd.

## Step 4: Kubelet Behavior Check

Create a pod and track lifecycle:

```bash
kubectl run bb-check --image=registry.k8s.io/pause:3.9 --restart=Never
kubectl describe pod bb-check
kubectl delete pod bb-check
```

Use events to describe scheduler vs kubelet responsibilities.
