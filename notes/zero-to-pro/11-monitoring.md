# Monitoring and Observability

Use metrics, logs, and alerts together. Metrics show trends, logs explain events, alerts drive action.

## The 4 golden signals

1. Latency
2. Traffic
3. Errors
4. Saturation

## Basic cluster checks

```bash
kubectl top nodes
kubectl top pods -A
kubectl get events --sort-by=.metadata.creationTimestamp
```

If `kubectl top` fails, install metrics-server.

## Application logging

```bash
kubectl logs deploy/web-api --all-pods=true --tail=200
kubectl logs -f deploy/web-api --all-pods=true
```

## Prometheus + Grafana quick install

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

## Alerting example idea

Trigger alert when 5xx error rate stays above 5% for 5 minutes.

## Dashboard essentials

- Pod restarts by namespace
- CPU and memory request/limit usage
- API latency (p50, p95, p99)
- Ingress error rate

## Incident triage flow

1. Alert fires.
2. Confirm impact in dashboard.
3. Inspect recent rollout/events.
4. Check pod logs and readiness failures.
5. Roll back if needed.

## Exercise alignment

- Primary: [exercises/07-observability/README.md](exercises/07-observability/README.md)
- Related reliability signals: [exercises/05-reliability/README.md](exercises/05-reliability/README.md)

Manifests to inspect:

- [exercises/07-observability/k8s/deployment.yaml](exercises/07-observability/k8s/deployment.yaml)
- [exercises/07-observability/k8s/deployment-broken.yaml](exercises/07-observability/k8s/deployment-broken.yaml)
- [exercises/07-observability/k8s/deployment-fixed.yaml](exercises/07-observability/k8s/deployment-fixed.yaml)

## Quick quiz

1. When should you use logs previous while debugging?
2. Which signal best indicates user-facing failures?
3. Why should events be checked along with logs?

Answer key:

1. When containers restart and prior instance output is needed.
2. Error rate (4xx/5xx depending on app expectations).
3. Events reveal scheduler/kubelet/controller actions not visible in app logs.
