# Final Capstone Lab: End-to-End Kubernetes Delivery

This capstone links the full learning track into one practical mission using the existing exercises in this repository.

## Mission

Build, operate, secure, observe, and package a production-style Kubernetes workload using the exercise modules in sequence.

## Completion criteria

- You complete phases 1 to 10 below in order.
- Every phase passes its acceptance criteria from the mapped exercise README.
- You produce a short capstone report with evidence commands and outputs.

## Runtime assumptions

- Namespace: k8s-lab
- Local cluster is healthy and kubectl context is correct
- Helm is installed

## Capstone flow by chapter and exercise

### Phase 1: Foundation deploy

- Chapter alignment: 01-introduction.md, 04-deployment-replicas.md, 04-kubernetes-services.md
- Exercise: exercises/01-first-deployment/README.md
- Evidence:
  - kubectl get deploy,svc,endpoints
  - kubectl rollout status deploy/web

### Phase 2: Safe delivery lifecycle

- Chapter alignment: 03-push-and-pull-images.md, 04-deployment-replicas.md
- Exercise: exercises/02-rollout-and-rollback/README.md
- Evidence:
  - kubectl rollout history deploy/web
  - kubectl rollout undo deploy/web

### Phase 3: Config and persistent state

- Chapter alignment: 06-persistant-states.md
- Exercise: exercises/03-config-and-state/README.md
- Evidence:
  - kubectl get configmap,secret,pvc
  - kubectl exec deploy/stateful-web -- sh -c "tail -n 5 /data/custom.log"

### Phase 4: Networking and ingress policy

- Chapter alignment: 04-kubernetes-services.md, 05-ingress.md
- Exercise: exercises/04-networking/README.md
- Evidence:
  - kubectl get endpoints net-web
  - allowed client succeeds, denied client fails

### Phase 5: Reliability under load

- Chapter alignment: 04-deployment-replicas.md, 11-monitoring.md
- Exercise: exercises/05-reliability/README.md
- Evidence:
  - kubectl describe pod -l app=reliable-web
  - kubectl get hpa reliable-web-hpa

### Phase 6: Security hardening

- Chapter alignment: 10-security.md
- Exercise: exercises/06-security/README.md
- Evidence:
  - kubectl auth can-i list pods --as=system:serviceaccount:k8s-lab:sa-reader -n k8s-lab
  - kubectl describe pod -l app=secure-web

### Phase 7: Observability and triage

- Chapter alignment: 11-monitoring.md
- Exercise: exercises/07-observability/README.md
- Evidence:
  - kubectl logs -l app=obs-web --tail=50
  - kubectl logs -l app=obs-web-broken --previous --tail=30

### Phase 8: Production release mechanics

- Chapter alignment: 07-helm.md, 02-writing-dockerfile.md
- Exercise: exercises/08-production/README.md
- Evidence:
  - helm lint helm/k8s-app
  - helm history app-staging -n k8s-lab
  - successful rollback from bad image tag

### Phase 9: Advanced workload patterns

- Chapter alignment: 09-full-stack.md
- Exercise: exercises/09-design-patterns/README.md
- Evidence:
  - init container runs first
  - sidecar and main container logs collected
  - job completion verified

### Phase 10: Platform extension model

- Chapter alignment: 13-gatekeepers-and-operators.md
- Exercise: exercises/10-crd-and-operator/README.md
- Evidence:
  - kubectl get crd widgets.training.k8s.io
  - kubectl get widget sample-widget -o yaml
  - explain why CRD without operator has no reconciliation behavior

## Optional architecture understanding phases

- exercises/11-kubernetes-building-blocks/README.md
- exercises/12-behind-the-scenes/README.md
- exercises/13-cloud-managed-kubernetes/README.md

These are recommended for deep platform reasoning and decision-making.

## Capstone report template

Create capstone-report.md with the following sections:

1. Environment and tool versions
2. Phase-by-phase evidence
3. Break/fix issues encountered and root causes
4. Security controls applied
5. Observability signals used during troubleshooting
6. Release and rollback summary
7. Final architecture and tradeoff notes

## Suggested execution command pack

Run each phase from its own exercise directory and capture output snippets:

```bash
kubectl get pods -A
kubectl get events --sort-by=.lastTimestamp
kubectl get deploy,svc,ingress,pvc,hpa -n k8s-lab
helm list -n k8s-lab
```

## Quick quiz

1. Which phase validates least-privilege access end-to-end?
2. Which phase proves production rollback readiness?
3. Why is phase sequencing important in this capstone?

Answer key:

1. Phase 6.
2. Phase 8.
3. Later phases assume stable foundations in deployment, networking, and state management.
