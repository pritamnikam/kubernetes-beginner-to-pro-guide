# Ingress: HTTP and HTTPS Entry Point

Ingress routes external HTTP(S) traffic to internal services.

## Prerequisite

Install an ingress controller (for example ingress-nginx). Ingress resources do nothing without a controller.

## Host and path routing example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  ingressClassName: nginx
  rules:
    - host: app.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-api
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-api
                port:
                  number: 80
```

## TLS example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress-tls
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - app.local
      secretName: app-local-tls
  rules:
    - host: app.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-api
                port:
                  number: 80
```

## Test locally

```bash
kubectl apply -f ingress.yaml
kubectl get ingress
kubectl describe ingress app-ingress
```

Add host mapping in local hosts file:

```txt
127.0.0.1 app.local
```

## Common issues

- `404` from ingress controller: path/host mismatch
- No external address: controller not running
- TLS handshake errors: wrong secret or certificate

## Exercise alignment

- Primary: [exercises/04-networking/README.md](exercises/04-networking/README.md)

Manifests to inspect:

- [exercises/04-networking/k8s/ingress.yaml](exercises/04-networking/k8s/ingress.yaml)
- [exercises/04-networking/k8s/service.yaml](exercises/04-networking/k8s/service.yaml)

## Quick quiz

1. Why does an Ingress resource require an Ingress controller?
2. What is the difference between host-based and path-based routing?
3. Where is TLS typically terminated in this pattern?

Answer key:

1. The controller implements and enforces Ingress rules.
2. Host routing matches domain names; path routing matches URL paths.
3. At the Ingress controller, before forwarding to Services.
