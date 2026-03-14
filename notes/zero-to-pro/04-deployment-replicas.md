# Deployments and Replicas

Deployments manage stateless workloads and keep your desired pod count running.

## Reliable deployment example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-api
spec:
  replicas: 3
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app: web-api
  template:
    metadata:
      labels:
        app: web-api
    spec:
      containers:
        - name: web-api
          image: myuser/flask-demo:1.0.0
          ports:
            - containerPort: 5000
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
          readinessProbe:
            httpGet:
              path: /
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: 5000
            initialDelaySeconds: 15
            periodSeconds: 20
      terminationGracePeriodSeconds: 30
```

## Core commands

```bash
kubectl apply -f deployment.yaml
kubectl get deploy,pods
kubectl scale deploy/web-api --replicas=5
kubectl rollout status deploy/web-api
```

## Rolling update example

```bash
kubectl set image deployment/web-api web-api=myuser/flask-demo:1.1.0
kubectl rollout status deployment/web-api
```

Rollback if needed:

```bash
kubectl rollout undo deployment/web-api
```

## Troubleshooting

- `CrashLoopBackOff`: app is crashing, inspect logs.
- Not Ready: probe path/port mismatch.
- No pods created: selector/labels mismatch.

Useful commands:

```bash
kubectl describe deploy web-api
kubectl logs deploy/web-api --all-pods=true --tail=100
```

## Exercise alignment

- Primary: [exercises/02-rollout-and-rollback/README.md](exercises/02-rollout-and-rollback/README.md)
- Related: [exercises/01-first-deployment/README.md](exercises/01-first-deployment/README.md)
- Related: [exercises/05-reliability/README.md](exercises/05-reliability/README.md)

Manifests to inspect:

- [exercises/02-rollout-and-rollback/k8s/deployment-v1.yaml](exercises/02-rollout-and-rollback/k8s/deployment-v1.yaml)
- [exercises/02-rollout-and-rollback/k8s/deployment-v2.yaml](exercises/02-rollout-and-rollback/k8s/deployment-v2.yaml)
- [exercises/05-reliability/k8s/deployment.yaml](exercises/05-reliability/k8s/deployment.yaml)

## Quick quiz

1. Which changes create a new Deployment revision?
2. What is the operational difference between rollout undo and set image?
3. Why are readiness probes critical for rolling updates?

Answer key:

1. Pod template changes, such as image/env/command/labels in template.
2. set image moves forward to a new revision; rollout undo returns to a previous revision.
3. They prevent traffic from reaching pods that are not ready yet.
