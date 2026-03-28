# URL Shortner

## Executive Summary
A globally distributed link shortening service capable of handling 10,000 requests per second with sub-10ms p99 latency.

This case study introduces every foundational K8s concept. You deploy a real API, configure Redis as a cache, expose it via Ingress with TLS, and watch autoscaling react to live load — all within a single namespace.

### NFRs
10k - RPS target
<10ms - p99 latency
3 - replicas
99.9% - SLA

What You Will Build: URL SHORTENER: 10k rps · stateless api
Difficulty Level: Beginner

### K8s Concepts You Will Master:
- Understand the Pod → ReplicaSet → Deployment ownership chain
- Differentiate ClusterIP, NodePort, and LoadBalancer Services
- Externalize config safely with ConfigMaps and Secrets
- Autoscale horizontally based on CPU and custom metrics
- Route external HTTPS traffic using Ingress + cert-manager
- Use Redis as a K8s StatefulSet with persistent storage


### System Architecture Overview

  ┌─────────────────────────────────────────────────────────┐
  │                   GKE CLUSTER                           │
  │                                                         │
  │  [Internet] ──→ [Cloud Load Balancer]                   │
  │                        │                                │
  │               [Ingress Controller]                      │
  │               nginx / GKE Ingress                       │
  │                        │                                │
  │          ┌─────────────┴──────────────┐                 │
  │          ↓                            ↓                 │
  │   [url-api Deployment]        [redirect-api]            │
  │   3 replicas → HPA(3-20)         1 replica              │
  │          │                            │                 │
  │          └──────────┬─────────────────┘                 │
  │                     ↓                                   │
  │           [Redis StatefulSet]  ←──── [ConfigMap]        │
  │           cache + counters           (REDIS_HOST etc)   │
  │                     │                                   │
  │           [PostgreSQL StatefulSet]                      │
  │           persistent url store                          │
  └─────────────────────────────────────────────────────────┘


## Building Blocks

### 📦Pod (Core)
The smallest deployable unit in Kubernetes. A Pod wraps one or more containers that share network and storage. The url-api runs as a single-container Pod managed by a Deployment.

WHY: Kubernetes manages Pods, not containers directly. Understanding Pod lifecycle (Pending→Running→Succeeded/Failed) is critical for debugging.

### 🔁 Deployment (Workload)
Declares the desired state for your app: which image, how many replicas, update strategy. The controller loop continuously reconciles actual vs desired state.

WHY: Never run bare Pods in production. Deployments give you rolling updates, rollback, and self-healing for free.

### 🌐 Service (Network)
A stable virtual IP (ClusterIP) that load-balances across all matching Pods. Even as Pods come and go, the Service IP stays constant.

WHY: Pod IPs are ephemeral. Services are the stable endpoint your other services and Ingress rely on.

### ⚙️ ConfigMap (Config)
Key-value store for non-sensitive configuration. Mount as env vars or files. The url-api reads REDIS_HOST, DB_PORT from here.

WHY: Separating config from code means one image runs in dev, staging, and prod — only the ConfigMap changes.

### 🔐 Secret (Config)
Like ConfigMap but base64-encoded (and ideally encrypted at rest via KMS). Stores DB passwords, API keys. Never bake secrets into Docker images.

WHY: Secrets can be rotated independently of code. With External Secrets Operator, they sync from GCP Secret Manager automatically.

### 📈 HPA (Autoscale)
Horizontal Pod Autoscaler watches CPU/memory metrics (or custom ones via KEDA) and scales replicas between minReplicas and maxReplicas.

WHY: Traffic is unpredictable. HPA lets the system scale out during viral spikes and scale in at night to save cost.

### 🚪Ingress (Network)
An L7 HTTP router. Maps hostnames and paths to backend Services. Works with nginx, Traefik, or GKE's built-in Ingress.

WHY: Without Ingress you'd need one LoadBalancer per service (expensive). Ingress shares one LB across all routes.

### 🗂️ Namespace (Isolation)
A virtual cluster within a cluster. Provides scope for names and resource quotas. All url-shortener resources live in the url namespace.

WHY: Namespaces prevent accidental cross-team interference and allow per-team ResourceQuotas and NetworkPolicies.

## Hads On

### 1. Phase 1 — First Deployment
Ship a container to Kubernetes and understand the fundamental primitives.
▶
- Install kubectl + gcloud CLI, configure kubeconfig
- Create GKE Autopilot cluster: `gcloud container clusters create-auto url-cluster`
- Write Deployment YAML: 1 replica of `url-api:latest`
- Apply it: `kubectl apply -f deployment.yaml`
- Watch pods start: `kubectl get pods -w`
- Scale manually: `kubectl scale deployment url-api --replicas=3`
- Describe a pod: `kubectl describe pod — read Events section`

### 2. Phase 2 — Services & Networking
Expose your app inside and outside the cluster.
▶
Create a ClusterIP Service for url-api (internal only)
Test with port-forward: `kubectl port-forward svc/url-api 8080:80`
Create a LoadBalancer Service — get external IP from EXTERNAL-IP column
Deploy Redis and expose via headless Service (clusterIP: None)
Verify DNS: `kubectl exec -it url-api -- nslookup redis`

### 3. Phase 3 — Config & Secrets
No hardcoded values. Ever.
▶
Create ConfigMap with REDIS_HOST, REDIS_PORT, DB_NAME
Create Secret with DB_PASSWORD (kubectl create secret generic)
Mount both as env vars in the Deployment spec
Rotate the DB_PASSWORD Secret without redeploying
Verify with: `kubectl exec -- printenv | grep REDIS`

### 4. Phase 4 — Ingress + TLS
Real HTTPS with automatic certificates.
▶
Install cert-manager: `kubectl apply -f cert-manager.yaml`
Create ClusterIssuer pointing to `Let's Encrypt`
Write Ingress resource: s.yourdomain.com → url-api:80
Annotate with cert-manager.io/cluster-issuer
Watch certificate issue: kubectl get certificate -w
Test: `curl -I https://s.yourdomain.com/abc123`

### 5. Phase 5 — Autoscaling
Handle a viral tweet hitting your service.
▶
Install Metrics Server (needed for HPA)
Write HPA: `minReplicas=2, maxReplicas=20, CPU=60%`
Run load test: `k6 run --vus 500 --duration 2m script.js`
Watch scale-out: `kubectl get hpa -w`
Add KEDA for Redis queue depth autoscaling (bonus)
Observe scale-in after load stops (5 min cooldown)