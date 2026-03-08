# Kubernetes Request Lifecycle (Text Diagram + Notes)

This note explains how a user HTTP request travels through Kubernetes from edge to container and back.

## Mental Model

There are two planes:

- Control plane: schedules and reconciles workloads.
- Data plane: moves real network traffic (your requests).

This document focuses on data plane request flow.

## End-to-End Flow

```text
[User/Browser]
      |
      v
[DNS resolves app host]
      |
      v
[External LB or NodePort]
      |
      v
[Ingress Controller Pod]
      |
      v
[Ingress Rule Match: host/path]
      |
      v
[Service (ClusterIP)]
      |
      v
[kube-proxy iptables/ipvs]
      |
      v
[Selected Pod IP]
      |
      v
[Container Port in Pod]
      |
      v
[Application Process]
      |
      v
[Response returns via same chain]
```

## Step-by-Step Notes

1. Client resolves domain name to ingress/load balancer endpoint.
2. Traffic reaches cluster edge (LoadBalancer service, NodePort, or local port-forward).
3. Ingress controller reads Ingress rules and selects backend Service.
4. Service selects healthy Pods through label selectors and endpoint list.
5. kube-proxy programs forwarding rules on nodes.
6. One Pod endpoint receives request.
7. Containerized app processes request and returns response.

## Where Requests Fail Most Often

- DNS host mismatch.
- Ingress rule host/path mismatch.
- Service selector not matching Pod labels.
- Pod not Ready (readiness probe failing).
- NetworkPolicy denying source Pod or namespace.
- Container listening on different port than Service targetPort.

## Fast Troubleshooting Map

```text
No response
  -> check ingress/lb reachability
  -> check service endpoints
  -> check pod readiness
  -> check app logs
```

Commands:

```bash
kubectl get ingress,svc,endpoints
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get networkpolicy
kubectl get events --sort-by=.lastTimestamp
```

## Control Plane Tie-In

Your request path depends on control-plane decisions made earlier:

- Scheduler placed Pod on node.
- Deployment/ReplicaSet kept desired replicas.
- Readiness probe marked Pod as traffic-eligible.

If those are wrong, data-plane routing can exist but still fail functionally.
