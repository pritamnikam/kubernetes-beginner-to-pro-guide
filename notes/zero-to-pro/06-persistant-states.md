# Persistent State in Kubernetes

Stateless workloads belong in Deployments. Databases and queues usually require StatefulSets and persistent storage.

## Storage building blocks

- PersistentVolume (PV): actual storage in cluster/cloud
- PersistentVolumeClaim (PVC): storage request by workload
- StorageClass: dynamic provisioning policy

## PVC example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## StatefulSet example

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:16
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
          ports:
            - containerPort: 5432
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 10Gi
```

## Validate persistence quickly

```bash
kubectl exec -it postgres-0 -- sh
echo "hello" > /var/lib/postgresql/data/test.txt
exit
kubectl delete pod postgres-0
kubectl exec -it postgres-0 -- cat /var/lib/postgresql/data/test.txt
```

If file remains, persistent storage is working.

## Best practices

- Use StatefulSet for stable network identity and storage.
- Back up database data outside the cluster.
- Use anti-affinity and multi-zone storage in production.

## Exercise alignment

- Primary: [exercises/03-config-and-state/README.md](exercises/03-config-and-state/README.md)

Manifests to inspect:

- [exercises/03-config-and-state/k8s/pvc.yaml](exercises/03-config-and-state/k8s/pvc.yaml)
- [exercises/03-config-and-state/k8s/deployment.yaml](exercises/03-config-and-state/k8s/deployment.yaml)
- [exercises/03-config-and-state/k8s/deployment-broken.yaml](exercises/03-config-and-state/k8s/deployment-broken.yaml)

## Quick quiz

1. Why is a StatefulSet preferred for databases over a Deployment?
2. What does a PVC being Bound indicate?
3. How do you prove data durability after a Pod restart?

Answer key:

1. StatefulSet preserves identity and storage mapping per Pod.
2. Requested storage has been matched/provisioned and attached logically.
3. Write data, recreate Pod, and confirm data still exists.
