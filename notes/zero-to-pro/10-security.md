# Kubernetes Security Essentials

Treat cluster security as layered defense: identity, runtime hardening, network policy, and secret handling.

## 1. Namespace isolation

```bash
kubectl create namespace dev
kubectl create namespace prod
```

Use separate namespaces per environment/team.

## 2. Least-privilege RBAC

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: dev
subjects:
  - kind: ServiceAccount
    name: ci-bot
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
```

## 3. Secrets and env injection

```bash
kubectl create secret generic db-secret --from-literal=password='change-me'
```

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

## 4. Pod security context

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 10001
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

## 5. Default deny network policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Then add explicit allow rules for app-to-db and ingress-to-app traffic.

## Security validation commands

```bash
kubectl auth can-i create pods --as system:serviceaccount:dev:ci-bot -n dev
kubectl get networkpolicy -A
kubectl describe pod <pod-name>
```

## Exercise alignment

- Primary: [exercises/06-security/README.md](exercises/06-security/README.md)
- Related network controls: [exercises/04-networking/README.md](exercises/04-networking/README.md)

Manifests to inspect:

- [exercises/06-security/k8s/serviceaccount.yaml](exercises/06-security/k8s/serviceaccount.yaml)
- [exercises/06-security/k8s/role.yaml](exercises/06-security/k8s/role.yaml)
- [exercises/06-security/k8s/secure-deployment.yaml](exercises/06-security/k8s/secure-deployment.yaml)

## Quick quiz

1. What is the difference between authentication and authorization in Kubernetes?
2. Why should RoleBinding target the smallest possible scope?
3. What runtime risk does allowPrivilegeEscalation false mitigate?

Answer key:

1. Authentication verifies identity; authorization decides permitted actions.
2. It enforces least privilege and limits blast radius.
3. It blocks processes from gaining higher privileges inside the container.
