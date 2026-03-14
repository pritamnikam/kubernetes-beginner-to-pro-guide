# Push and Pull Container Images

## Why this matters

Kubernetes nodes pull images from a registry. If tagging or auth is wrong, pods will fail with `ImagePullBackOff`.

## Tagging strategy

Use immutable tags for releases.

- Good: `myapp:1.4.2`, `myapp:git-2f8a1c4`
- Risky: `myapp:latest`

## End-to-end example (Docker Hub)

```bash
docker login
docker build -t myuser/flask-demo:1.0.0 .
docker push myuser/flask-demo:1.0.0
```

## Deployment that pulls from registry

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-demo
  template:
    metadata:
      labels:
        app: flask-demo
    spec:
      containers:
        - name: app
          image: myuser/flask-demo:1.0.0
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 5000
```

## Private registry access

Create pull secret:

```bash
kubectl create secret docker-registry regcred \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>
```

Reference in pod spec:

```yaml
spec:
  imagePullSecrets:
    - name: regcred
```

## Debug checklist

```bash
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Look for auth errors, wrong image names, or missing tags.

## Exercise alignment

- Primary: [exercises/02-rollout-and-rollback/README.md](exercises/02-rollout-and-rollback/README.md)
- Related: [exercises/01-first-deployment/README.md](exercises/01-first-deployment/README.md)
- Related: [exercises/08-production/README.md](exercises/08-production/README.md)

Broken image drills in this repo:

- [exercises/01-first-deployment/k8s/deployment-broken.yaml](exercises/01-first-deployment/k8s/deployment-broken.yaml)
- [exercises/02-rollout-and-rollback/k8s/deployment-broken.yaml](exercises/02-rollout-and-rollback/k8s/deployment-broken.yaml)

## Quick quiz

1. Why are immutable image tags preferred over latest?
2. What does ImagePullBackOff usually indicate?
3. When do you need imagePullSecrets?

Answer key:

1. They make deployments reproducible and auditable.
2. Pull/auth/tag issues while fetching image.
3. When pulling from a private registry.
