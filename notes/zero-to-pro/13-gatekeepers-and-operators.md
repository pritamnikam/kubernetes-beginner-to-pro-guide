# Gatekeeper Policies and Operators

This chapter covers two advanced platform skills:

- Policy enforcement with OPA Gatekeeper
- Day-2 automation with Operators

## Gatekeeper in practice

RBAC controls who can act. Gatekeeper controls what is allowed.

### Example policy goal

Reject pods without an `owner` label.

Constraint:

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: pod-must-have-owner
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
  parameters:
    labels:
      - key: owner
```

Test outcome:

- Pod without `owner` label: denied
- Pod with `owner` label: admitted

## CRDs and Operators

- CRD extends Kubernetes API with new resource types.
- Operator is a custom controller that reconciles those resources.

Typical operator use cases:

- Databases (Postgres, MySQL)
- Certificates (cert-manager)
- Messaging systems (Kafka)

## Operator example (cert-manager)

Create a certificate declaratively and let operator renew it automatically.

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: app-cert
spec:
  secretName: app-tls
  issuerRef:
    name: letsencrypt
    kind: ClusterIssuer
  dnsNames:
    - app.example.com
```

## Platform engineering checklist

1. Enforce required labels and resource limits.
2. Block privileged containers.
3. Use operators for complex stateful systems.
4. Keep policy and operator manifests in Git for auditability.

## Exercise alignment

- Primary: [exercises/10-crd-and-operator/README.md](exercises/10-crd-and-operator/README.md)
- Platform decision extension: [exercises/13-cloud-managed-kubernetes/README.md](exercises/13-cloud-managed-kubernetes/README.md)

Manifests to inspect:

- [exercises/10-crd-and-operator/k8s/crd.yaml](exercises/10-crd-and-operator/k8s/crd.yaml)
- [exercises/10-crd-and-operator/k8s/widget-sample.yaml](exercises/10-crd-and-operator/k8s/widget-sample.yaml)

## Quick quiz

1. What does a CRD give you that native resources do not?
2. Why is a controller required after registering a CRD?
3. How is policy enforcement different from RBAC?

Answer key:

1. Custom API resource types for domain-specific objects.
2. Reconciliation logic is needed to turn desired custom state into real resources.
3. RBAC controls who can act; policy controls what configurations are allowed.
