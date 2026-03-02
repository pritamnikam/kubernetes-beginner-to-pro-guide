# Week 1 — Kubernetes Foundations (Zero to Productive)

This week builds your core Kubernetes mental model and command fluency.

## Learning Outcomes

By the end of Week 1, you should be able to:

1. Explain how desired state and reconciliation work.
2. Create and inspect Pods and Deployments.
3. Expose an app with a Service.
4. Troubleshoot common failures using `kubectl`.

## Concept First (Before Commands)

Kubernetes is a control-plane system:

- You declare intent in YAML.
- Controllers compare **desired state** vs **actual state**.
- Kubernetes continuously reconciles differences.

Key objects for this week:

- **Pod**: smallest deployable unit; one or more containers.
- **Deployment**: manages replicas and rolling updates for stateless apps.
- **Service**: stable virtual IP + DNS for accessing Pods.

## Prerequisites

- Local cluster: Minikube, Kind, or Docker Desktop Kubernetes.
- `kubectl` installed.
- A working context shown by:

```bash
kubectl config current-context
kubectl get nodes
```

If Docker Hub pulls fail with `unexpected EOF` in your environment, use images from `registry.k8s.io` for all exercises.

## Study Cadence (3 Sessions)

## Session 1 — Cluster and Pod Basics (60–90 min)

### Learn

- Cluster architecture: control plane, worker node.
- Pod lifecycle: Pending → Running → Succeeded/Failed.

### Do

```bash
kubectl get nodes
kubectl get namespaces
kubectl run demo-pod --image=registry.k8s.io/pause:3.9 --restart=Never
kubectl get pods -o wide
kubectl describe pod demo-pod
kubectl logs demo-pod
```

### Verify

- Pod reaches `Running`.
- You can explain events from `describe` output.

---

## Session 2 — Deployments and Services (60–90 min)

### Learn

- Why Deployment over direct Pod creation.
- ReplicaSet relationship.
- Service abstraction and selectors.

### Do

Work through [Exercise 01](../exercises/01-first-deployment/README.md).

### Verify

- Deployment has 2/2 ready replicas.
- Service endpoints are populated.
- App is reachable via port-forward.

---

## Session 3 — Troubleshooting Drill (60–90 min)

### Learn

A simple debug order:

1. `kubectl get`
2. `kubectl describe`
3. `kubectl logs`
4. `kubectl get events --sort-by=.lastTimestamp`

### Do

Run the broken manifest from Exercise 01 and fix it.

### Verify

- You identify root cause quickly.
- You patch or re-apply corrected YAML.
- Workload returns to healthy state.

## Mastery Check (End of Week)

You pass Week 1 when you can do all of this from memory:

- Create a namespace and set context for it.
- Apply Deployment + Service from YAML.
- Rollout status and history checks.
- Diagnose one `ImagePullBackOff` or label mismatch issue.

## Real-World Error Patterns (from practice)

- `unknown flag` (example: `--namespce`, `--mage`) means command parsing failed before Kubernetes API call. Re-run with corrected flag spelling.
- `ImagePullBackOff` can mean bad tag, registry issue, or transient network pull failure.
- Debug image pulls with:

```bash
kubectl describe pod <pod-name>
kubectl get events --sort-by=.lastTimestamp
```

- If tag is wrong, delete and recreate pod with a known-good image (for this lab use `registry.k8s.io/pause:3.9`).

## What’s Next

After Week 1, continue to Phase 2 in [README](../README.md):

- scaling and rollouts,
- service types,
- deployment strategies.
