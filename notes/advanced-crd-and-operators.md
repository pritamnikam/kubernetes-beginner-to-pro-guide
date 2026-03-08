# Advanced: CRDs and Operators

## Mental Model

- CRD adds a new API type to Kubernetes.
- Custom Resource is an instance of that type.
- Operator is a controller that reconciles desired state for that resource.

## How It Works

1. Register CRD.
2. Create custom object.
3. API server stores it in etcd.
4. Operator watches for changes.
5. Operator reconciles by creating/updating native resources.

Without step 4, CRD objects are data only. No automation occurs.

## When to Use

- Stateful systems with domain-specific operations.
- Complex app lifecycle automation (backup, failover, upgrades).
- Repeated human runbooks that should be encoded.

## Operator Safety Checklist

- Idempotent reconcile loops.
- Backoff and retry strategy.
- Clear status conditions.
- Finalizers for safe cleanup.

Reference hands-on: [Exercise 10](../exercises/10-crd-and-operator/README.md)
