# Week 3 — Config, Secrets, and Persistent State

This week maps to Exercise 03 and teaches separation of code, config, and data.

## Core Concepts

- `ConfigMap` is for non-sensitive runtime config.
- `Secret` is for sensitive values.
- `PVC` gives stable storage across Pod restarts.
- Stateful behavior comes from storage binding, not from Pod identity alone.

## Why This Matters

Containers are ephemeral. Real applications need safe config injection and durable data.

## Practice Flow

1. Create ConfigMap, Secret, and PVC.
2. Deploy app consuming all three.
3. Validate config/secret wiring from logs.
4. Write data, delete Pod, confirm data persists.
5. Run break/fix for `CreateContainerConfigError`.

Reference lab: [Exercise 03](../exercises/03-config-and-state/README.md)

## Essential Commands

```bash
kubectl get configmap,secret,pvc
kubectl rollout status deploy/stateful-web
kubectl logs -l app=stateful-web --tail=30
kubectl exec deploy/stateful-web -- sh -c "tail -n 5 /data/custom.log"
kubectl describe pod -l app=stateful-web-broken
```

## Debug Checklist

1. If Pod is pending, check PVC binding.
2. If container config fails, inspect missing secret/config keys.
3. Never print secret values to logs.

## Mastery Signals

- You can choose ConfigMap vs Secret correctly.
- You can explain why PVC preserves data after Pod recreation.
- You can fix a bad `secretKeyRef` quickly.
