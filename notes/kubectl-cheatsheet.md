# kubectl Quick Sheet (Week 1)

## Context and Cluster

```bash
kubectl config get-contexts
kubectl config current-context
kubectl get nodes
```

## Namespace

```bash
kubectl create namespace k8s-lab
kubectl config set-context --current --namespace=k8s-lab
kubectl get ns
```

## Apply and Inspect

```bash
kubectl apply -f <file-or-folder>
kubectl get all
kubectl get pods -o wide
kubectl describe pod <pod-name>
```

## Logs and Events

```bash
kubectl logs <pod-name>
kubectl logs -f <pod-name>
kubectl get events --sort-by=.lastTimestamp
```

## Deployments

```bash
kubectl get deploy
kubectl rollout status deploy/<name>
kubectl rollout history deploy/<name>
kubectl scale deploy/<name> --replicas=3
```

## Service Access

```bash
kubectl get svc
kubectl get endpoints
kubectl port-forward svc/<service-name> 8080:80
```

## Cleanup

```bash
kubectl delete -f <file-or-folder>
kubectl delete namespace k8s-lab
```
