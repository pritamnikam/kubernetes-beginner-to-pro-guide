# Checklist

Here it is, Coach—the **Kubernetes Pro-Lab Cheat Sheet**. This covers every command and workflow we’ve touched, from building your first image to scaling an automated empire.

---

## 🏗️ 1. Building & Packaging (Docker)

| Task | Command |
| --- | --- |
| **Build Image** | `docker build -t my-app:local .` |
| **List Images** | `docker images` |
| **Remove Image** | `docker rmi my-app:local` |
| **Run Locally** | `docker run -p 5000:5000 my-app:local` |

---

## 🚢 2. The Kubernetes CLI (`kubectl`)

| Task | Command |
| --- | --- |
| **Apply YAML** | `kubectl apply -f lab.yaml` |
| **Get Resources** | `kubectl get pods` (or `svc`, `deploy`, `hpa`, `all`) |
| **Watch Changes** | `kubectl get pods -w` |
| **Describe Details** | `kubectl describe pod [name]` |
| **View Logs** | `kubectl logs -f deployment/python-lab-app` |
| **Delete Resource** | `kubectl delete -f lab.yaml` |
| **Interactive Shell** | `kubectl exec -it [pod-name] -- sh` |

---

## 🚦 3. Routing & Connectivity

| Task | Command |
| --- | --- |
| **Port Forward** | `kubectl port-forward svc/[svc-name] 8080:80` |
| **Check Services** | `kubectl get svc` |
| **NodePort URL** | Access via `http://localhost:[nodePort]` |

---

## 📈 4. Scaling & Automation (HPA/VPA)

| Task | Command |
| --- | --- |
| **Manual Scale** | `kubectl scale deployment/python-lab-app --replicas=10` |
| **Create HPA** | `kubectl autoscale deploy [name] --cpu-percent=50 --min=2 --max=10` |
| **Check HPA** | `kubectl get hpa` |
| **View Resources** | `kubectl top pods` (Requires Metrics Server) |

---

## 📦 5. Package Management (Helm)

| Task | Command |
| --- | --- |
| **Add Repo** | `helm repo add [name] [url]` |
| **Install Chart** | `helm install [release-name] [chart-name]` |
| **List Releases** | `helm list` |
| **Rollback** | `helm rollback [release-name] [revision-number]` |
| **Uninstall** | `helm uninstall [release-name]` |

---

## 🔑 6. Security & Configuration

| Task | Command |
| --- | --- |
| **Create Secret** | `kubectl create secret generic [name] --from-literal=key=value` |
| **Create ConfigMap** | `kubectl create configmap [name] --from-literal=key=value` |
| **Check Secrets** | `kubectl get secrets` |

---

## 🚨 7. Troubleshooting Strategy

1. **`kubectl get pods`**: Are they running? (Check for `CrashLoopBackOff` or `Pending`).
2. **`kubectl describe pod [name]`**: Look at the "Events" at the bottom. This is where you find why a pod failed to start.
3. **`kubectl logs [name]`**: What does the app's internal code say?
4. **`kubectl get events --sort-by=.metadata.creationTimestamp`**: A chronological timeline of everything happening in the cluster.

---

### 🎓 What's your next adventure?

You’ve mastered the core. Whenever you're ready to tackle something specific—like **CI/CD pipelines**, **Service Mesh**, or **Multi-Cloud clusters**—you know where to find me.

**Would you like me to explain how to prepare for the CKA (Certified Kubernetes Administrator) exam now that you have these foundations?**