# 🎟️ TICKET BOOKING
50k concurrent · race-safe · Intermediate

## Executive Summary
A highly available ticket reservation system that handles massive concurrent writes without overselling, using Kubernetes Jobs for async processing.

Ticket booking is deceptively hard — race conditions, double-sells, unhappy customers. This study teaches async processing with Jobs, graceful disruption handling, and zero-downtime rolling updates for a system that cannot afford a moment's downtime.

- 50k: concurrent users
- 0: oversells
- 5min: reservation TTL
- <100ms: booking p99

### What You Will Build
TICKET BOOKING: 50k concurrent · race-safe

### Difficulty Level
Intermediate

### K8s Concepts You Will Master
- Run one-off tasks and batch jobs with Kubernetes Jobs
- Schedule recurring operations with CronJobs
- Protect availability during node maintenance with PodDisruptionBudgets (PDB)
- Segment east-west traffic with NetworkPolicies
- Implement graceful shutdown with preStop hooks and terminationGracePeriod
- Blue/green deployments for zero booking downtime

### System Architecture Overview

  ┌───────────────────────────────────────────────────────────┐
  │                    GKE CLUSTER                            │
  │                                                           │
  │  [User] ──→ [Ingress] ──→ [booking-api Deployment]        │
  │                                    │                      │
  │                    ┌───────────────┼───────────────┐      │
  │                    ↓               ↓               ↓      │
  │            [PostgreSQL]      [Kafka Topic]    [Redis]     │
  │            seat inventory    booking-events   seat lock   │
  │                               │                           │
  │                ┌──────────────┼──────────────┐            │
  │                ↓              ↓              ↓            │
  │         [payment-worker] [email-worker] [analytics]       │
  │         Deployment        Deployment     CronJob          │
  │                                                           │
  │  [CronJob: expire-reservations] runs every 5 minutes      │
  │  [Job: seat-import] one-time bulk CSV import              │
  └───────────────────────────────────────────────────────────┘

## Building Blocks

### ⚡Job (Workload)
Runs one or more Pods to completion. Retries on failure (backoffLimit). Supports parallel execution (parallelism). Used for one-time tasks like DB migrations or data imports.

WHY: Some workloads are not long-running servers. Jobs give Kubernetes-native run-to-completion semantics with retries, timeouts, and parallelism.

### ⏰ CronJob (Workload)
Creates Jobs on a cron schedule. Used to expire unpaid reservations every 5 minutes. Uses standard cron syntax (*/5 * * * *).

WHY: Without CronJobs you'd need an external scheduler or a long-running loop in your app. CronJobs are declarative, auditable, and cluster-managed.

### 🛡️PodDisruptionBudget (Availability)
Limits how many Pods can be voluntarily disrupted at once (node drain, cluster upgrade). Set minAvailable: 2 to always keep 2 booking-api pods running.

WHY: Without PDB, a `kubectl drain` during a GKE upgrade could terminate all your pods simultaneously, causing downtime.

### 🔒NetworkPolicy (Security)
A firewall for Pod-to-Pod traffic. Default deny-all, then explicitly allow booking-api → postgres, payment-worker → kafka. Works at L3/L4.

WHY: By default, all pods can talk to all other pods. NetworkPolicy enforces least-privilege networking — a compromised pod can't reach the payment system.

### 🩺ReadinessProbe (Health)
Kubernetes only sends traffic to a Pod once the readinessProbe passes. During startup or when degraded, the pod is temporarily removed from Service endpoints.

WHY: Critical during rolling updates. Without it, traffic hits new pods before they've connected to Kafka and Redis, causing booking errors.

### 🩹LivenessProbe (Health)
Restarts a Pod if the liveness check fails N times. Catches deadlocks and stuck goroutines that don't crash but can't process requests.

WHY: A deadlocked booking process silently stops accepting bookings. LivenessProbe detects and self-heals this automatically.

### 🔄preStop Hook (Lifecycle)
A command run inside the container before Kubernetes sends SIGTERM. Used to finish in-flight booking requests before the pod shuts down.

WHY: Without preStop, a mid-booking transaction gets killed. The hook lets the pod drain gracefully: sleep 5 lets the LB remove it from rotation first.

### 📊ResourceQuota (Policy)
Limits total resource consumption in a namespace: max 20 pods, 10 CPU, 20Gi memory. Prevents one team from starving another.

WHY: Without quotas, a runaway Job could consume all cluster resources. Quotas are namespace-level capacity management.

## Hands-on Phases

### Phase 1 — Health Probes & Graceful Shutdown
Make Kubernetes aware of your app lifecycle.
▶
- Add readinessProbe: httpGet /ready — only ready when Kafka connected
- Add livenessProbe: httpGet /healthz — restart if DB pool exhausted
- Add preStop hook: sleep 5 to let LB drain before SIGTERM
- Set terminationGracePeriodSeconds: 30
- Simulate deadlock: watch liveness restart the pod automatically

### Phase 2 — Async Jobs
Process payments and notifications without blocking the booking response.
▶
- Deploy Kafka as a StatefulSet with 3 brokers
- Write payment-worker as a Deployment consuming kafka topic
- Create a one-off Job: import seats from CSV file
- Create a CronJob: expire unpaid reservations every 5 minutes
- Simulate job failure: watch backoffLimit retries in kubectl get jobs

### Phase 3 — Zero Downtime Deploys
Deploy booking-api v2 without losing a single booking.
▶
- Set RollingUpdate: maxSurge=1, maxUnavailable=0
- Create PodDisruptionBudget: minAvailable=2
- While k6 runs 1000 VUs, deploy v2: kubectl set image ...
- Verify 0 HTTP errors during rollout
- Test node drain: kubectl drain --ignore-daemonsets
- Watch PDB prevent more than 1 pod eviction at once

### Phase 4 — NetworkPolicy Lockdown
Strict east-west traffic control.
▶
- Apply default-deny-all NetworkPolicy in booking namespace
- Allow booking-api → postgres (port 5432)
- Allow booking-api → kafka (port 9092)
- Allow payment-worker → kafka only
- Verify: kubectl exec random-pod -- nc -zv postgres 5432 — should fail
- Debug: use kubectl exec + nc / nmap / curl to trace connectivity

## Commnads

### CronJob — Expire Reservations

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: expire-reservations
  namespace: booking
spec:
  schedule: "*/5 * * * *"           # every 5 minutes
  concurrencyPolicy: Forbid          # don't overlap if previous still running
  startingDeadlineSeconds: 60        # skip if can't start within 60s
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      backoffLimit: 2                # retry failed jobs up to 2 times
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: expiry-worker
            image: gcr.io/proj/booking-worker:latest
            command: ["/app/expire-reservations"]
            env:
            - name: DB_HOST
              valueFrom:
                secretKeyRef:
                  name: booking-secrets
                  key: db-host
```

### PodDisruptionBudget
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: booking-pdb
  namespace: booking
spec:
  # Always keep at least 2 pods running
  # Even during GKE node upgrades or kubectl drain
  minAvailable: 2
  selector:
    matchLabels:
      app: booking-api

# Note: set maxUnavailable instead if you prefer:
# maxUnavailable: 1  — allow at most 1 pod down at once
```

### NetworkPolicy — Default Deny + Allow

```yaml
# Step 1: Block ALL ingress and egress by default
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: booking
spec:
  podSelector: {}          # applies to ALL pods in namespace
  policyTypes:
  - Ingress
  - Egress

---
# Step 2: Allow booking-api to reach postgres only
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-booking-to-postgres
  namespace: booking
spec:
  podSelector:
    matchLabels:
      app: booking-api
  policyTypes: [Egress]
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgres
    ports:
    - protocol: TCP
      port: 5432
```


## Commands

### Deploy with zero-downtime check

```bash
# Deploy v2 while load test runs
kubectl set image deployment/booking-api \
  api=gcr.io/proj/booking-api:v2.0.0 -n booking

# Watch rolling update — should see 0% errors
kubectl rollout status deployment/booking-api -n booking

# Drain a node (simulates GKE upgrade)
kubectl drain <node-name> \
  --ignore-daemonsets \
  --delete-emptydir-data

# PDB prevents > 1 pod evicted simultaneously
kubectl get pdb booking-pdb -n booking
```

### Debug NetworkPolicy
```bash
# Test connectivity (should fail after default-deny)
kubectl run nettest --image=busybox --rm -it -- \
  nc -zv postgres 5432

# Verify NetworkPolicy is applied
kubectl get networkpolicies -n booking

# Describe to see pod selector details
kubectl describe networkpolicy default-deny-all -n booking
```