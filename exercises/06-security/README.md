# Exercise 06 — ServiceAccount, RBAC, and Pod Security

## Concept First

Phase 6 is about reducing blast radius.

- **ServiceAccount** gives workload identity in Kubernetes.
- **RBAC** controls what that identity can do.
- **Pod security context** reduces container runtime risk (non-root, no privilege escalation, limited Linux capabilities).

Core idea: secure by default, grant only what is required.

## Objective

1. Create a dedicated ServiceAccount.
2. Grant least-privilege read access with Role/RoleBinding.
3. Verify permissions using `kubectl auth can-i`.
4. Deploy a workload with hardened security context.
5. Run break/fix drill for misconfigured RoleBinding.

## Acceptance Criteria

- ServiceAccount `sa-reader` exists.
- `kubectl auth can-i` returns `yes` for allowed actions and `no` for disallowed actions.
- `secure-web` deployment is healthy.
- Pod spec shows hardened security context settings.
- Broken RoleBinding drill is diagnosed and fixed.

## Prerequisites

- Active namespace context: `k8s-lab`
- Working cluster and `kubectl`

## Step 1: Apply ServiceAccount and RBAC

```bash
kubectl apply -f k8s/serviceaccount.yaml
kubectl apply -f k8s/role.yaml
kubectl apply -f k8s/rolebinding.yaml
kubectl get sa,role,rolebinding
```

## Step 2: Verify Least-Privilege Permissions

Allowed:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:k8s-lab:sa-reader -n k8s-lab
kubectl auth can-i get services --as=system:serviceaccount:k8s-lab:sa-reader -n k8s-lab
```

Disallowed (example):

```bash
kubectl auth can-i delete pods --as=system:serviceaccount:k8s-lab:sa-reader -n k8s-lab
```

Expected:

- Allowed checks return `yes`.
- Disallowed check returns `no`.

## Step 3: Deploy Pod using ServiceAccount

```bash
kubectl apply -f k8s/rbac-check-pod.yaml
kubectl get pod rbac-check
kubectl describe pod rbac-check
```

Check `Service Account: sa-reader` in describe output.

## Step 4: Deploy Hardened Web Workload

```bash
kubectl apply -f k8s/secure-deployment.yaml
kubectl rollout status deploy/secure-web
kubectl get pods -l app=secure-web
kubectl describe pod -l app=secure-web
```

Verify in `describe` output:

- `runAsNonRoot: true`
- `allowPrivilegeEscalation: false`
- `readOnlyRootFilesystem: true`
- `capabilities.drop: ["ALL"]`
- `seccompProfile.type: RuntimeDefault`

Note: this workload keeps root filesystem read-only but mounts writable ephemeral storage (`emptyDir`) at `/certs`, `/var/lib/nginx`, `/var/cache/nginx`, and `/run` for runtime-generated files.

## Step 5: Break/Fix Drill (RoleBinding Subject Typo)

Break it:

```bash
kubectl delete rolebinding reader-binding
kubectl apply -f k8s/rolebinding-broken.yaml
kubectl auth can-i list pods --as=system:serviceaccount:k8s-lab:sa-reader -n k8s-lab
```

Expected: returns `no` due to wrong subject name.

Fix it:

```bash
kubectl delete rolebinding reader-binding-broken
kubectl apply -f k8s/rolebinding.yaml
kubectl auth can-i list pods --as=system:serviceaccount:k8s-lab:sa-reader -n k8s-lab
```

Expected: returns `yes` again.

## Cleanup (Optional)

```bash
kubectl delete -f k8s/secure-deployment.yaml --ignore-not-found
kubectl delete -f k8s/rbac-check-pod.yaml --ignore-not-found
kubectl delete -f k8s/rolebinding-broken.yaml --ignore-not-found
kubectl delete -f k8s/rolebinding.yaml --ignore-not-found
kubectl delete -f k8s/role.yaml --ignore-not-found
kubectl delete -f k8s/serviceaccount.yaml --ignore-not-found
```

## Common Mistakes

- Binding broad roles (`cluster-admin`) for simple read tasks.
- Not testing effective permissions with `kubectl auth can-i`.
- Assuming Pod security without setting explicit securityContext fields.
