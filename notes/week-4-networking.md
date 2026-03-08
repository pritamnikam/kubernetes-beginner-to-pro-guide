# Week 4 — Service Discovery, Ingress, and Traffic Policy

This week maps to Exercise 04 and teaches in-cluster and ingress traffic flow.

## Core Concepts

- Service gives stable DNS and virtual IP over changing Pod IPs.
- Ingress handles HTTP routing to Services.
- NetworkPolicy restricts Pod-to-Pod traffic with label selectors.

## Why This Matters

Most incidents in distributed apps are connectivity and policy mismatches.

## Practice Flow

1. Deploy app and Service.
2. Validate DNS from client Pods.
3. Apply deny-all ingress policy.
4. Add allow-list policy for one client.
5. Run broken Service selector drill.

Reference lab: [Exercise 04](../exercises/04-networking/README.md)

## Essential Commands

```bash
kubectl get svc,endpoints net-web
kubectl exec net-client-allowed -- wget -qO- http://net-web
kubectl apply -f k8s/networkpolicy-deny-all.yaml
kubectl apply -f k8s/networkpolicy-allow-from-allowed-client.yaml
kubectl get networkpolicy
```

## Debug Checklist

1. Empty endpoints usually means selector mismatch.
2. If policy seems ignored, verify Pod labels and CNI policy support.
3. Ingress tests require an installed controller.

## Mastery Signals

- You can explain why Service works even when Pods rotate.
- You can prove allow/deny behavior with two clients.
- You can fix selector mismatches from endpoints output.
