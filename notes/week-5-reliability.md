# Week 5 — Probes, Resources, and Autoscaling

This week maps to Exercise 05 and teaches runtime resilience.

## Core Concepts

- Startup probe protects slow startup from premature restarts.
- Readiness probe controls traffic eligibility.
- Liveness probe restarts unhealthy containers.
- Requests/limits influence scheduling and runtime stability.
- HPA scales replicas based on metrics.

## Why This Matters

Reliable systems fail gracefully and self-heal under load.

## Practice Flow

1. Deploy baseline with startup/readiness/liveness probes.
2. Verify requests and limits in Pod spec.
3. Create HPA and generate load (if metrics server exists).
4. Break probe port and restore healthy probe config.

Reference lab: [Exercise 05](../exercises/05-reliability/README.md)

## Essential Commands

```bash
kubectl describe pod -l app=reliable-web
kubectl get hpa reliable-web-hpa
kubectl top pods
kubectl apply -f k8s/deployment-broken.yaml
kubectl describe pod -l app=reliable-web-broken
```

## Debug Checklist

1. Failing readiness means no traffic.
2. Failing liveness means restarts.
3. Use events to see probe failure details.
4. HPA without metrics-server will not scale.

## Mastery Signals

- You can choose probe type per failure mode.
- You can set practical resource envelopes.
- You can explain why scaling happened or did not happen.
