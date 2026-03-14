# Zero to Pro: Kubernetes Learning Track

This folder is a hands-on path from container basics to production-grade Kubernetes operations.

## What you will learn

1. Package apps with Docker.
2. Store and version images in a registry.
3. Run workloads with Deployments.
4. Expose apps with Services and Ingress.
5. Persist state with volumes and StatefulSets.
6. Secure workloads and access.
7. Monitor reliability and performance.
8. Package and manage releases with Helm.
9. Complete an end-to-end capstone that ties all modules together.

## Prerequisites

- Docker Desktop (Kubernetes enabled) or kind/minikube
- kubectl
- Helm
- Basic Linux/terminal commands

## Quick architecture mental model

- Docker image: immutable app package
- Pod: running instance of one or more containers
- Deployment: manages stateless pod replicas
- Service: stable network endpoint for pods
- Ingress: HTTP/HTTPS routing at cluster edge
- StatefulSet + PVC: stable identity and storage

## First working example

Apply this deployment and service to verify your cluster is healthy:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-nginx
  template:
    metadata:
      labels:
        app: demo-nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: demo-nginx
spec:
  selector:
    app: demo-nginx
  ports:
    - port: 80
      targetPort: 80
  type: NodePort
```

Commands:

```bash
kubectl apply -f demo.yaml
kubectl get pods,svc
kubectl port-forward svc/demo-nginx 8080:80
```

Open http://localhost:8080.

## Study strategy

- Read one file.
- Run all commands in that file.
- Break one manifest intentionally and debug it.
- Move to next file only after you can explain the previous one in your own words.

## Exercise alignment

- Primary: [exercises/01-first-deployment/README.md](exercises/01-first-deployment/README.md)
- Next: [exercises/11-kubernetes-building-blocks/README.md](exercises/11-kubernetes-building-blocks/README.md)
- Deep dive: [exercises/12-behind-the-scenes/README.md](exercises/12-behind-the-scenes/README.md)

## Quick quiz

1. What is the main difference between a Deployment and a Pod?
2. Why is a Service needed if Pods are already running?
3. Which component routes external HTTP traffic to internal Services?

Answer key:

1. Deployment manages and reconciles replicas of Pods.
2. Pod IPs are ephemeral; Service provides stable discovery and load balancing.
3. Ingress (with an installed Ingress controller).
