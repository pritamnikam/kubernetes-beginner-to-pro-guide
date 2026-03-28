# 🛒E-COMMERCE
microservices · service mesh · Advanced

## Executive Summary
A cloud-native e-commerce platform decomposed into 6 independently deployable microservices, managed by Istio service mesh and deployed via GitOps.

Microservices at scale require infrastructure beyond basic K8s. Istio adds observability, security (mTLS), and traffic management that is impossible to implement in application code alone. This study prepares you for real-world platform engineering.

- 6: microservices
- mTLS: all comms
- 10%: canary traffic
- GitOps: deploy model

### What You Will Build
E-COMMERCE: microservices · service mesh

### Difficulty Level
Advanced

### K8s Concepts You Will Master
- Decompose a monolith into independently deployable microservices
- Install and configure Istio service mesh with sidecar injection
- Enforce mTLS between all services using PeerAuthentication
- Implement canary deployments via VirtualService traffic splitting
- Implement GitOps: every cluster change via a Git commit
- Use Argo Rollouts for automated progressive delivery with analysis

### System Architecture Overview
  ┌───────────────────────────────────────────────────────────────┐
  │  GKE CLUSTER + ISTIO MESH                                     │
  │                                                               │
  │  [Browser] ──→ [Istio Gateway] ──→ [VirtualService router]    │
  │                                           │                   │
  │         ┌──────────────┬─────────────┬────┴────────┐          │
  │         ↓              ↓             ↓             ↓          │
  │   [frontend-v1]  [product-svc]  [cart-svc]  [order-svc]       │
  │   [frontend-v2]       │              │          │             │
  │   90% → v1            └──────────────┴──────────┘             │
  │   10% → v2                           │                        │
  │                                 [payment-svc]                 │
  │                                      │                        │
  │                              [Stripe / GCP Pay]               │
  │                                                               │
  │  Every pod has Envoy sidecar injected automatically           │
  │  All traffic: mTLS encrypted + traced + metered               │
  │                                                               │
  │  ArgoCD watches github.com/org/ecom-k8s                       │
  │  Git push → ArgoCD detects drift → auto-sync to cluster       │
  └───────────────────────────────────────────────────────────────┘


## Building Blocks

### 🕸️Istio Mesh (Platform)
A service mesh that injects an Envoy sidecar proxy into every Pod. The sidecar intercepts all traffic, providing observability, mTLS, retries, timeouts, and circuit breaking — without changing application code.

WHY: At 6+ services, debugging a failed request across the chain is nearly impossible without distributed tracing and traffic metrics that Istio provides out of the box.

### 🚦VirtualService (Istio)
Istio's traffic routing layer. Splits traffic by percentage (canary), matches by headers (A/B testing), sets retries and timeouts per route.

WHY: Enables safe rollouts. Send 10% of traffic to frontend-v2. If error rate stays low, shift to 50%, then 100% — all without redeployment.

### 🏷️DestinationRule (Istio)
Defines traffic policy for a service: load balancing algorithm, mTLS mode, connection pool limits, outlier detection (circuit breaking).

WHY: Circuit breaking at the mesh level prevents a slow downstream service from cascading failures. If payment-svc fails 5 times, stop sending traffic for 30s.

### 🔐PeerAuthentication (Security)
Istio policy enforcing mTLS between services. In STRICT mode, all plaintext traffic is rejected — every service must present a valid certificate.

WHY: In a microservices mesh, internal traffic can still be intercepted. mTLS means even a compromised internal pod can't read traffic between other services.

### 🌀Argo Rollout (Delivery)
A CRD that replaces Deployment for advanced delivery: canary, blue/green with automated analysis. Integrates with Prometheus to auto-rollback if error rate rises.

WHY: Manual canary (shifting VirtualService weights) requires a human watching dashboards. Argo Rollouts automates the entire progressive delivery pipeline.

### 🗂️ArgoCD Application (GitOps)
An ArgoCD CRD that declares: "this cluster should match this Git repo path." ArgoCD continuously syncs the cluster to Git — the repo is the source of truth.

WHY: kubectl apply by hand is not auditable or reproducible. With GitOps, every change has a PR, approval, and commit hash. Rollback = git revert.

### 📊ServiceMonitor (Observability)
A Prometheus CRD that tells Prometheus which pods to scrape for metrics. Selector matches pods by label. Interval = 15s by default.

WHY: Without metrics, you're flying blind. ServiceMonitor is the glue between your app's /metrics endpoint and Prometheus dashboards.

### 🌐Gateway (Istio)
The Istio equivalent of K8s Ingress. Manages inbound/outbound traffic at the mesh boundary. More powerful than Ingress: supports TCP, gRPC, WebSocket.

WHY: Istio Gateway + VirtualService replaces K8s Ingress in a mesh environment, giving you L7 routing with full Istio traffic management features.

## Hands-on Phases

### Phase 1 — Microservice Decomposition
Split the monolith into independent services.
▶
- Deploy 6 services in their own namespaces: frontend, product, cart, order, payment, notification
- Each service has its own Deployment, Service, ConfigMap, ServiceAccount
- Inter-service communication via ClusterIP DNS: `product-svc.product.svc.cluster.local`
- Write a dependency diagram — identify circular deps
- Verify all services start and can reach each other

### Phase 2 — Istio Installation
Add the mesh layer to all services.
▶
- Install Istio: `istioctl install --set profile=demo`
- Enable sidecar injection: `kubectl label namespace product istio-injection=enabled`
- Restart all pods to inject sidecars
- Verify: `kubectl describe pod product-api | grep istio-proxy`
- Open Kiali UI: `istioctl dashboard kiali`
- Watch live traffic graph — see real request flows

### Phase 3 — mTLS + Circuit Breaking
Secure and protect all service communication.
▶
- Apply PeerAuthentication `STRICT` mode to all namespaces
- Verify plaintext rejected: `kubectl exec -- curl http://product-svc` (should fail)
- Add DestinationRule with outlier detection for payment-svc
- Simulate payment failure: kill payment pod during checkout
- Watch circuit breaker open in Kiali

### Phase 4 — Canary Deploy + GitOps
Ship frontend-v2 safely and automate everything.
▶
- Install ArgoCD, connect to your GitHub repo
- Deploy frontend-v2 alongside v1
- Write VirtualService: 90% v1, 10% v2
- Monitor error rates in Prometheus
- Shift to 50/50, then 100% v2
- Install Argo Rollouts, migrate frontend Deployment to Rollout CRD
- Configure analysis: auto-rollback if error rate > 1%

## YAML References

### Istio VirtualService — Canary

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: frontend
  namespace: frontend
spec:
  hosts:
  - frontend-svc
  http:
  - name: canary-split
    route:
    - destination:
        host: frontend-svc
        subset: v1           # DestinationRule defines these subsets
      weight: 90             # 90% of traffic to stable version
    - destination:
        host: frontend-svc
        subset: v2
      weight: 10             # 10% canary traffic
    retries:
      attempts: 3
      perTryTimeout: 2s
    timeout: 10s

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: frontend
  namespace: frontend
spec:
  host: frontend-svc
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 100
  subsets:
  - name: v1
    labels:
      version: v1            # matches pods with label version=v1
  - name: v2
    labels:
      version: v2
```

### Argo Rollout — Progressive Delivery

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: frontend
  namespace: frontend
spec:
  replicas: 5
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
  strategy:
    canary:
      canaryService: frontend-canary-svc
      stableService: frontend-stable-svc
      analysis:
        templates:
        - templateName: success-rate    # Prometheus query
        startingStep: 2
      steps:
      - setWeight: 10          # 10% traffic to canary
      - pause: {duration: 5m}  # wait 5 minutes
      - setWeight: 30
      - pause: {duration: 5m}
      - setWeight: 60
      - pause: {duration: 5m}
      - setWeight: 100         # full rollout
```      

### ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ecommerce
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/org/ecom-k8s.git
    targetRevision: main         # track main branch
    path: k8s/                   # all YAML lives here
  destination:
    server: https://kubernetes.default.svc
    namespace: frontend
  syncPolicy:
    automated:
      prune: true                # delete resources removed from Git
      selfHeal: true             # re-apply if someone kubectl edits manually
    syncOptions:
    - CreateNamespace=true
```

## Commnads

### Install Istio + enable mesh
```bash
# Install Istio control plane
istioctl install --set profile=demo -y

# Enable injection for all app namespaces
for ns in frontend product cart order payment; do
  kubectl label namespace $ns istio-injection=enabled
  kubectl rollout restart deployment -n $ns
done

# Verify sidecars are injected
kubectl get pods -n product -o jsonpath='{.items[*].spec.containers[*].name}'

# Open dashboards
istioctl dashboard kiali
istioctl dashboard grafana
istioctl dashboard jaeger
```


### GitOps workflow
```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f \
  https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get initial admin password
kubectl get secret argocd-initial-admin-secret \
  -n argocd -o jsonpath='{.data.password}' | base64 -d

# Port-forward ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Now: push any change to Git → ArgoCD auto-syncs to cluster
git add k8s/ && git commit -m "feat: bump frontend to v2"
git push origin main
# ArgoCD detects drift within 3 minutes and applies changes
```