# Writing a Better Dockerfile

## Goals

- Small image size
- Fast rebuilds
- Reproducible dependencies
- Secure runtime defaults

## Example app files

`requirements.txt`

```txt
flask==3.1.0
gunicorn==23.0.0
```

`app.py`

```python
from flask import Flask
app = Flask(__name__)

@app.get("/")
def health():
  return {"status": "ok"}
```

## Production-oriented Dockerfile

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Install dependencies first for layer caching.
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .

EXPOSE 5000

# Run as non-root.
RUN useradd -m appuser
USER appuser

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

## Build and test locally

```bash
docker build -t flask-demo:v1 .
docker run --rm -p 5000:5000 flask-demo:v1
curl http://localhost:5000/
```

## Common mistakes

- Using `latest` tags in production
- Copying entire project before dependency install (slower rebuilds)
- Running app as root
- Baking secrets into image layers

## Improvement challenge

Create `.dockerignore` with at least:

```txt
.git
__pycache__
*.pyc
.env
node_modules
```

## Exercise alignment

- Primary: [exercises/08-production/README.md](exercises/08-production/README.md)
- Related drill (image hygiene and pull behavior): [exercises/01-first-deployment/README.md](exercises/01-first-deployment/README.md)

Use these files while practicing:

- [exercises/08-production/helm/k8s-app/templates/deployment.yaml](exercises/08-production/helm/k8s-app/templates/deployment.yaml)
- [exercises/08-production/helm/k8s-app/values.yaml](exercises/08-production/helm/k8s-app/values.yaml)

## Quick quiz

1. Why should dependencies be installed before copying full source code?
2. Why avoid running containers as root?
3. What is the risk of baking secrets into image layers?

Answer key:

1. It improves layer-cache reuse and speeds rebuilds.
2. It reduces blast radius if the container is compromised.
3. Secrets can leak through image history and registries.
