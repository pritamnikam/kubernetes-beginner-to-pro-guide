# Helm: Package and Manage Kubernetes Releases

Helm helps you templatize manifests and manage release history.

## Create a chart

```bash
helm create web-api
tree web-api
```

Important files:

- `Chart.yaml`: chart metadata
- `values.yaml`: default settings
- `templates/*.yaml`: Kubernetes templates

## Template example

In `templates/deployment.yaml`, image values should come from `values.yaml`:

```yaml
containers:
  - name: {{ .Chart.Name }}
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
    imagePullPolicy: {{ .Values.image.pullPolicy }}
```

`values.yaml`:

```yaml
replicaCount: 2
image:
  repository: myuser/flask-demo
  tag: "1.0.0"
  pullPolicy: IfNotPresent
```

## Deploy and upgrade

```bash
helm install web-api ./web-api
helm upgrade web-api ./web-api --set image.tag=1.1.0
helm history web-api
```

Rollback:

```bash
helm rollback web-api 1
```

## Environment overrides

Use separate values files:

```bash
helm upgrade --install web-api-dev ./web-api -f values-dev.yaml
helm upgrade --install web-api-prod ./web-api -f values-prod.yaml
```

## Safety checks before deploy

```bash
helm lint ./web-api
helm template web-api ./web-api > rendered.yaml
kubectl apply --dry-run=server -f rendered.yaml
```

## Exercise alignment

- Primary: [exercises/08-production/README.md](exercises/08-production/README.md)

Manifests to inspect:

- [exercises/08-production/helm/k8s-app/Chart.yaml](exercises/08-production/helm/k8s-app/Chart.yaml)
- [exercises/08-production/helm/k8s-app/values-staging.yaml](exercises/08-production/helm/k8s-app/values-staging.yaml)
- [exercises/08-production/helm/k8s-app/values-prod.yaml](exercises/08-production/helm/k8s-app/values-prod.yaml)

## Quick quiz

1. Why render templates with helm template before deploying?
2. What is the value of separate staging and prod values files?
3. What problem does helm rollback solve during incidents?

Answer key:

1. It exposes final manifests early for validation.
2. It enforces environment-specific configuration without chart duplication.
3. It restores a known-good release revision quickly.
