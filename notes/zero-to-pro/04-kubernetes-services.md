# Kubernetes Services

Pods are ephemeral. Services provide stable networking in front of changing pod IPs.

## Service types

- ClusterIP: internal-only (default)
- NodePort: exposes service on each node IP and fixed port
- LoadBalancer: cloud-managed external load balancer

## Practical example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-api
spec:
  selector:
    app: web-api
  ports:
    - name: http
      port: 80
      targetPort: 5000
  type: ClusterIP
```

## Verify service endpoints

```bash
kubectl get svc web-api
kubectl get endpoints web-api
kubectl describe svc web-api
```

If endpoints are empty, selector labels do not match pod labels.

## Temporary local access patterns

Port-forward service:

```bash
kubectl port-forward svc/web-api 8080:80
```

NodePort example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-api-nodeport
spec:
  selector:
    app: web-api
  type: NodePort
  ports:
    - port: 80
      targetPort: 5000
      nodePort: 30080
```

## Best practices

- Keep service selectors simple and explicit.
- Use readiness probes so unready pods do not receive traffic.
- Prefer Ingress over many `LoadBalancer` services for HTTP apps.

## Exercise alignment

- Primary: [exercises/04-networking/README.md](exercises/04-networking/README.md)
- Related: [exercises/01-first-deployment/README.md](exercises/01-first-deployment/README.md)

Manifests to inspect:

- [exercises/01-first-deployment/k8s/service.yaml](exercises/01-first-deployment/k8s/service.yaml)
- [exercises/04-networking/k8s/service.yaml](exercises/04-networking/k8s/service.yaml)
- [exercises/04-networking/k8s/service-broken.yaml](exercises/04-networking/k8s/service-broken.yaml)

## Quick quiz

1. What happens when a Service selector matches zero Pods?
2. When should you use ClusterIP versus NodePort?
3. Why does a Service continue working even when Pod IPs change?

Answer key:

1. Service gets no endpoints and traffic fails.
2. ClusterIP for internal traffic; NodePort for simple external/local access.
3. kube-proxy routes to current endpoints selected by labels.
