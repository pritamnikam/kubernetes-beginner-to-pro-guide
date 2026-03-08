# Exercise 13 — Cloud Managed Kubernetes Analysis Lab

## Objective

Build practical platform judgment across AKS, EKS, and GKE.

## Part A: Comparison Matrix

Create a table with these categories:

- Control plane management
- Upgrade experience
- Node pool model
- IAM integration
- Networking model
- Cost drivers
- Security features
- Ecosystem fit

## Part B: Scenario Mapping

For each scenario, choose AKS/EKS/GKE and justify:

1. Enterprise Microsoft-heavy stack with Entra ID.
2. AWS-native platform team with IAM-first controls.
3. Data/ML-heavy stack integrated with Google services.

## Part C: Production Readiness Checklist

Draft a cloud-agnostic checklist including:

- Multi-zone node pools
- Backup/restore strategy
- Policy enforcement (OPA/Gatekeeper/Kyverno)
- Secrets and identity strategy
- Upgrade and rollback plan
- Cost guardrails

## Deliverable

Create `managed-k8s-decision.md` in this folder with your matrix and decisions.
