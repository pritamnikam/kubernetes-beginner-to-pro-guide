# Cleanup and Reset

Cleanup prevents wasted local resources and avoids leftover objects affecting future labs.

## Remove app resources

```bash
kubectl delete -f lab.yaml
kubectl delete ingress --all
kubectl delete svc --all
kubectl delete deploy --all
kubectl delete statefulset --all
```

## Remove Helm releases

```bash
helm list -A
helm uninstall <release-name> -n <namespace>
```

## Remove namespaces created for labs

```bash
kubectl delete namespace dev
kubectl delete namespace prod
kubectl delete namespace monitoring
```

## Remove local images (optional)

```bash
docker image ls
docker image rm myuser/flask-demo:1.0.0
docker image prune -f
```

## Verify clean state

```bash
kubectl get all -A
kubectl get pvc -A
kubectl get pv
```

## Last-resort reset

If cluster state is broken during experimentation, reset Kubernetes in Docker Desktop settings.

## Exercise alignment

- Applies to all modules from [exercises/01-first-deployment/README.md](exercises/01-first-deployment/README.md) through [exercises/10-crd-and-operator/README.md](exercises/10-crd-and-operator/README.md)
- Production cleanup examples: [exercises/08-production/README.md](exercises/08-production/README.md)

## Quick quiz

1. Why is deleting manifests preferred over deleting Pods directly?
2. What should you verify after cleanup besides Pods?
3. When is full cluster reset justified?

Answer key:

1. It removes desired state and prevents controllers from recreating resources.
2. PVC/PV, namespaces, Helm releases, and lingering Services/Ingress.
3. When local state is irrecoverably inconsistent during learning.
