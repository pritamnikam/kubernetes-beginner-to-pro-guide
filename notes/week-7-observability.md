# Week 7 — Observability and Incident Triage

This week maps to Exercise 07 and teaches structured troubleshooting.

## Core Concepts

- Logs answer "what happened in the process".
- Describe/events answer "what Kubernetes did".
- Metrics answer "what pressure existed".
- `--previous` logs are critical for crash loops.

## Why This Matters

Without observability discipline, debugging becomes trial and error.

## Practice Flow

1. Deploy healthy target and confirm Service availability.
2. Practice `logs`, `logs -f`, and lifecycle correlation with events.
3. Run broken deployment and capture previous logs.
4. Apply fixed manifest and verify recovery.

Reference lab: [Exercise 07](../exercises/07-observability/README.md)

## Essential Commands

```bash
kubectl logs -l app=obs-web --tail=50
kubectl logs -f -l app=obs-web
kubectl describe pod -l app=obs-web
kubectl logs -l app=obs-web-broken --previous --tail=30
kubectl get events --sort-by=.lastTimestamp | findstr /I obs-web
```

## Debug Checklist

1. Check rollout and pod state.
2. Check current logs.
3. If restarting, check previous logs.
4. Check describe/events.
5. Apply smallest safe fix and verify.

## Mastery Signals

- You can identify root cause without opening YAML first.
- You can explain why previous logs differ from current logs.
- You can restore broken workload quickly and verify stability.
