# Full-Stack Example on Kubernetes

This chapter stitches frontend, backend, and database into one deployable topology.

## Reference architecture

- Frontend: Deployment + Service
- Backend API: Deployment + Service
- Database: StatefulSet + headless Service + PVC
- Ingress: routes external traffic to frontend/backend

## Minimal deployment order

1. Apply namespace and secrets.
2. Apply database manifests and wait ready.
3. Apply backend with DB connection env vars.
4. Apply frontend.
5. Apply ingress.

## Backend deployment example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: myuser/backend:1.0.0
          env:
            - name: DATABASE_URL
              value: postgres://postgres:$(DB_PASSWORD)@postgres:5432/app
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: password
          ports:
            - containerPort: 8080
```

## Frontend service example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
```

## End-to-end verification

```bash
kubectl get pods
kubectl get svc
kubectl get ingress
kubectl logs deploy/backend --tail=50
```

## Suggested exercises

1. Scale backend from 2 to 6 replicas and watch load balancing.
2. Roll backend image from `1.0.0` to `1.1.0` and rollback.
3. Delete one backend pod and confirm self-healing.

## Exercise alignment

- Build blocks: [exercises/03-config-and-state/README.md](exercises/03-config-and-state/README.md), [exercises/04-networking/README.md](exercises/04-networking/README.md), [exercises/05-reliability/README.md](exercises/05-reliability/README.md), [exercises/06-security/README.md](exercises/06-security/README.md)
- Release flow: [exercises/08-production/README.md](exercises/08-production/README.md)
- Patterns: [exercises/09-design-patterns/README.md](exercises/09-design-patterns/README.md)

## Quick quiz

1. Which components in this architecture should be stateless vs stateful?
2. Why should database credentials be injected from Secret rather than image env defaults?
3. What is the minimum rollout check you should run after backend image updates?

Answer key:

1. Frontend/backend are stateless; database is stateful.
2. It avoids credential leakage and enables secure rotation.
3. kubectl rollout status on the backend Deployment.
