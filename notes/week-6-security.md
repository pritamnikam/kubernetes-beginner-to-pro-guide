# Week 6 — Identity, Authorization, and Pod Hardening

This week maps to Exercise 06 and teaches least-privilege security.

## Core Concepts

- ServiceAccount is workload identity.
- RBAC controls what that identity can do.
- Pod and container security context reduce runtime risk.
- Read-only root filesystem often needs explicit writable mounts.

## Why This Matters

Most severe incidents become bigger because blast radius was not limited.

## Practice Flow

1. Create ServiceAccount and least-privilege Role/RoleBinding.
2. Validate permissions with `kubectl auth can-i`.
3. Deploy hardened workload with non-root and restricted privileges.
4. Run RoleBinding typo break/fix drill.

Reference lab: [Exercise 06](../exercises/06-security/README.md)

## Essential Commands

```bash
kubectl auth can-i list pods --as=system:serviceaccount:k8s-lab:sa-reader -n k8s-lab
kubectl auth can-i delete pods --as=system:serviceaccount:k8s-lab:sa-reader -n k8s-lab
kubectl describe pod -l app=secure-web
kubectl logs deploy/secure-web --tail=50
```

## Debug Checklist

1. If permissions fail, inspect RoleBinding subject namespace/name.
2. If hardened Pod crashes, inspect writable path requirements from logs.
3. Keep root filesystem read-only, but mount minimal writable `emptyDir` paths.

## Mastery Signals

- You can prove allowed and denied actions with `can-i`.
- You can explain each securityContext setting and trade-off.
- You can fix hardened runtime crashes without weakening security posture.
