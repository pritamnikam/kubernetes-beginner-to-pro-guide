# Set Up a Local Kubernetes Lab

This setup works well on Windows with Docker Desktop.

## Option A: Docker Desktop Kubernetes (fastest)

1. Open Docker Desktop.
2. Go to Settings > Kubernetes.
3. Enable Kubernetes and restart.

Verify:

```bash
kubectl cluster-info
kubectl get nodes
```

## Option B: kind (repeatable multi-cluster testing)

```bash
winget install Kubernetes.kind
kind create cluster --name k8s-lab
kubectl cluster-info --context kind-k8s-lab
```

## Install essential tools (Windows)

```bash
winget install Kubernetes.kubectl
winget install Helm.Helm
```

Verify:

```bash
kubectl version --client
helm version
```

## Smoke test app

```bash
kubectl create deployment hello --image=nginx:1.27
kubectl expose deployment hello --port=80 --type=NodePort
kubectl get pods,svc
kubectl port-forward svc/hello 8080:80
```

Open http://localhost:8080.

## Optional tools

- k9s for terminal UI
- Lens/OpenLens for graphical cluster view

## Troubleshooting

- `kubectl` cannot connect: check current context with `kubectl config current-context`.
- Pods stuck in `ImagePullBackOff`: verify internet access and image name.
- No metrics in `kubectl top`: install metrics-server.

## Exercise alignment

- Start here: [exercises/01-first-deployment/README.md](exercises/01-first-deployment/README.md)
- Then: [exercises/02-rollout-and-rollback/README.md](exercises/02-rollout-and-rollback/README.md)

## Quick quiz

1. What command confirms your current kube context?
2. Why is a smoke test useful before real exercises?
3. Which local setup is easiest for first-time learners on Windows?

Answer key:

1. kubectl config current-context.
2. It validates cluster connectivity and basic scheduling/networking.
3. Docker Desktop Kubernetes.
