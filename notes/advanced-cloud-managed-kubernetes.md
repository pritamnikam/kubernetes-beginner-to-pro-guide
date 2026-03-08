# Advanced: Cloud Managed Kubernetes

## What Managed Kubernetes Gives You

- Managed control plane operations.
- Integrated IAM, load balancers, and storage.
- Upgrade tooling and node pool lifecycle controls.

## Popular Managed Platforms

- AKS (Azure Kubernetes Service)
- EKS (Amazon Elastic Kubernetes Service)
- GKE (Google Kubernetes Engine)

## Comparison Dimensions

- Upgrade model and release channels.
- Networking model (CNI behavior, IP consumption).
- Identity integration and RBAC model.
- Cost model: control plane, nodes, egress, storage.
- Security defaults and policy tooling.

## Production Guidance

- Use multi-node pools for workload isolation.
- Enable workload identity and avoid long-lived secrets.
- Define PodDisruptionBudgets for critical apps.
- Separate staging and prod clusters or namespaces with strict policy.

Reference hands-on: [Exercise 13](../exercises/13-cloud-managed-kubernetes/README.md)
