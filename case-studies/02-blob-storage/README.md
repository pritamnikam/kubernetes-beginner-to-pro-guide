# BLOB STORAGE
s3-like · multi-tenant · Intermediate

## Executive Summary

A multi-tenant, S3-compatible blob storage service with per-team isolation, workload-based GCP auth, and persistent storage backed by GCS.

This study forces you to think about state. Databases and storage systems cannot be treated like stateless APIs — you need StatefulSets, persistent volumes, and careful upgrade strategies. RBAC teaches you the K8s security model.

- 100TB: storage capacity
- <50ms: upload latency
- 4: tenants
- 99.99%: durability

### What You Will Build
BLOB STORAGE: s3-like · multi-tenant

### Difficulty Level
Intermediate

### K8s Concepts You Will Master
- Difference between Deployment and StatefulSet (ordered, stable identity)
- PersistentVolume, PersistentVolumeClaim, and StorageClass lifecycle
- Workload Identity: authenticate to GCP APIs without key files
- RBAC: Roles, ClusterRoles, RoleBindings, ServiceAccounts
- Namespace isolation for multi-tenant environments
- Headless Services for stable DNS to StatefulSet pods

### System Architecture Overview

  ┌────────────────────────────────────────────────────────────┐
  │                     GKE CLUSTER                            │
  │                                                            │
  │  [API Client] ──→ [Ingress] ──→ [blob-api Deployment]      │
  │                                        │                   │
  │                         ┌──────────────┼──────────────┐    │
  │                         ↓              ↓              ↓    │
  │                  [GCS Bucket]   [PostgreSQL]    [Redis]    │
  │                  (objects)      StatefulSet     (presign)  │
  │                  via Workload   postgres-0       cache     │
  │                  Identity       postgres-1                 │
  │                                      │                     │
  │                               [PVC: ssd-20Gi]              │
  │                               volumeClaimTemplate          │
  │                                                            │
  │  Namespaces:  team-a ns │ team-b ns │ team-c ns            │
  │  Each with own RoleBinding → can only see own resources    │
  └────────────────────────────────────────────────────────────┘


## Building Blocks

### 💾 StatefulSet (Workload)
Like Deployment but pods get stable, ordered names (postgres-0, postgres-1) and their own PVCs via volumeClaimTemplates. Pods start and stop in order.

WHY: Databases need stable identities. postgres-0 is always the primary. postgres-1 knows to replicate from postgres-0. This predictability is impossible with Deployments.

### 📀 PersistentVolume (Storage)
A piece of cluster-level storage provisioned from a cloud disk (GCP Persistent Disk, AWS EBS). Lives independently of any Pod.

WHY: Pod storage is ephemeral. When a pod dies, its data dies too — unless it's mounted from a PersistentVolume.

### 📋 PersistentVolumeClaim (Storage)
A Pod's request for a PersistentVolume. Specifies size and access mode. The StorageClass dynamically provisions a matching PV.

WHY: Decouples the Pod from the specific disk. The cluster fulfills the claim — dev and prod can use different StorageClasses.

### 🏷️StorageClass (Storage)
Defines the type of storage provisioner (pd-ssd, pd-standard) and reclaim policy. Allows dynamic provisioning on demand.

WHY: Without StorageClass, an admin must manually create every PV. With it, claims are fulfilled automatically.

### 🆔 ServiceAccount (Security)
An identity for Pods (not humans). Every Pod runs as a ServiceAccount. Used for RBAC permissions and Workload Identity GCP auth.

WHY: Pods need to call K8s API or GCP APIs. ServiceAccounts are their identity. Never use the default SA — always create a dedicated one.

### 🛡️ RBAC Role (Security)
Namespace-scoped rules: which verbs (get, list, create, delete) on which resources (pods, secrets). Combined with RoleBinding to grant to a ServiceAccount.

WHY: Principle of least privilege. A blob-api SA should only read Secrets, not delete Deployments. RBAC enforces this at the API server.

### 🔑 Workload Identity (GCP)
Links a K8s ServiceAccount to a GCP Service Account. The Pod gets a GCP identity without any key file mounted. Credentials are short-lived and auto-rotated.

WHY: Storing GCP SA key files in Secrets is a security risk. Workload Identity eliminates the key entirely — no leak surface.

### 📡Headless Service (Network)
A Service with clusterIP: None. DNS resolves to individual Pod IPs instead of a virtual IP. Clients see postgres-0.postgres, postgres-1.postgres.

WHY: Required for StatefulSets. Clients (like primary-replica setups) need to address individual pods by stable hostname.

## Hadns-on Phase

### Phase 1 — StatefulSet Fundamentals
Deploy Postgres as a StatefulSet and understand ordered pod management.
▶
- Write a StatefulSet for PostgreSQL with volumeClaimTemplates
- Observe pod naming: postgres-0, postgres-1, postgres-2
- Delete postgres-1 — watch it recreate with same name and PVC
- Create a Headless Service: clusterIP: None
- Verify DNS from another pod: nslookup postgres-0.postgres


### Phase 2 — Storage Deep Dive
Understand PVs, PVCs and StorageClasses.
▶
- Create a StorageClass backed by pd-ssd
- Create a PVC: 20Gi, ReadWriteOnce
- Mount it into a Pod at /data
- Write a file, delete the Pod, recreate — verify file persists
- Change Reclaim Policy to Retain and test what happens on PVC delete

### Phase 3 — Workload Identity
Zero-key-file GCP authentication.
▶
- Create a GCP Service Account with roles/storage.objectAdmin
- Create a K8s ServiceAccount in blob namespace
- Annotate KSA with iam.gke.io/gcp-service-account
- Bind GCP SA to K8s SA via IAM policy binding
- Deploy a pod as that KSA and run: gsutil ls gs://my-bucket

### Phase 4 — Multi-Tenant RBAC
Strict namespace isolation for teams.
▶
- Create namespaces: team-a, team-b, team-c
- Create a Role in each: can read/write own Secrets
- Create RoleBinding: bind team-a SA to team-a Role
- Verify: `kubectl auth can-i get secrets -n team-b --as system:serviceaccount:team-a:default`
- Create a ClusterRole for the blob-api to read across namespaces
- Audit using: `kubectl get rolebindings -A`

## YAML

### StatefulSet (PostgreSQL)
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: blob
spec:
  serviceName: "postgres"    # must match headless Service name
  replicas: 2
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      serviceAccountName: blob-ksa    # Workload Identity SA
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_DB
          value: blobstore
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: pg-data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:                # each pod gets its own PVC
  - metadata:
      name: pg-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: ssd            # your custom StorageClass
      resources:
        requests:
          storage: 20Gi
```

### RBAC — Role & RoleBinding
```yaml
# Role: scoped to team-a namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: team-a-role
  namespace: team-a
rules:
- apiGroups: [""]
  resources: ["secrets", "configmaps"]
  verbs: ["get", "list", "create", "update"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list"]              # can read but NOT delete

---
# RoleBinding: grant Role to ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-a-binding
  namespace: team-a
subjects:
- kind: ServiceAccount
  name: team-a-sa
  namespace: team-a
roleRef:
  kind: Role
  name: team-a-role
  apiGroup: rbac.authorization.k8s.io
```

### Workload Identity annotation
```yaml
# Step 1: K8s ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: blob-ksa
  namespace: blob
  annotations:
    # This annotation links K8s SA to GCP SA
    iam.gke.io/gcp-service-account: blob-gsa@my-project.iam.gserviceaccount.com

---
# Step 2: GCP CLI commands to complete the binding
# (run outside cluster)
#
# gcloud iam service-accounts add-iam-policy-binding \
#   blob-gsa@my-project.iam.gserviceaccount.com \
#   --role roles/iam.workloadIdentityUser \
#   --member "serviceAccount:my-project.svc.id.goog[blob/blob-ksa]"
```

## Commands

### StorageClass + PVC
```bash
# Create SSD StorageClass
kubectl apply -f - <<'EOF'
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ssd
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-ssd
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
EOF

# Create PVC
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: blob-data
spec:
  storageClassName: ssd
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 20Gi
EOF
```

### Verify RBAC permissions
```bash
# Test: can team-a SA access team-b secrets?
kubectl auth can-i get secrets \
  -n team-b \
  --as system:serviceaccount:team-a:team-a-sa
# Expected: no

# List all RoleBindings cluster-wide
kubectl get rolebindings -A

# Impersonate SA and try real kubectl command
kubectl get pods -n team-b \
  --as system:serviceaccount:team-a:team-a-sa
```