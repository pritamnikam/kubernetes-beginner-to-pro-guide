# HPA

In the Google Kubernetes team, we call these the **"Dynamic Duo"** of production. One makes your app **flexible** (settings), and the other makes it **elastic** (scaling).

---

## 🛠️ ConfigMaps & Secrets: The "No-Rebuild" Rule

**The Problem:** You built your Docker image with a database URL pointing to your "Dev" database. Now you want to move to "Production."

* **The Wrong Way:** Change the code and rebuild the Docker image. (Slow and dangerous).
* **The Pro Way:** Use **ConfigMaps** and **Secrets**.

### Analogy: The Car and the GPS

Think of your **Docker Image** as a **Car**. You don't build a new car just because you want to drive to a different city. Instead, you change the **Destination** in the **GPS**.

* **ConfigMap:** The GPS destination (Public info: Database URLs, Feature flags).
* **Secret:** The Garage Door Opener (Private info: Passwords, API Keys, SSL Certs).

### How it works:

You inject these into your Pod as **Environment Variables** or **Files**. When the Pod starts, it reads the "GPS" and knows exactly where to go.

---

## 📈 Horizontal Pod Autoscaling (HPA)

**The Problem:** At 2:00 PM, you have 10,000 users. At 2:00 AM, you have 10.

* If you keep 50 pods running all night, you **waste money**.
* If you only keep 2 pods running, your site **crashes** during the day.

### Analogy: The Grocery Store Checkout

Imagine a grocery store.

1. **Metric:** The store manager watches the length of the lines (CPU/Memory usage).
2. **The Trigger:** If a line has more than 5 people, the manager pages another cashier (HPA creates a new Pod).
3. **The Scale Down:** When the store is empty, the manager tells the extra cashiers to go home (HPA deletes the Pods).

### The "Pro" YAML for HPA:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: python-lab-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # If CPU hits 70%, add more pods!

```

---

## 🛡️ Problem Statement: The "Thundering Herd"

**The Problem:** Your HPA sees a spike and adds 10 pods. Those 10 pods all try to connect to your Database at the exact same millisecond. Your Database crashes.

**The Solution:** In your **ConfigMap**, you might configure "Connection Pooling" or "Backoff" settings. This shows how Scaling and Configuration work together to keep the system stable.

---

## 🏁 Summary for your Pro Notes:

* **ConfigMaps:** Store your `APP_COLOR` or `API_URL`.
* **Secrets:** Store your `DB_PASSWORD` (Encoded in Base64).
* **HPA:** Your "Auto-Pilot" that saves money at night and saves your reputation during a sale.

**Would you like to try adding an HPA to your local Python Lab app to see it try to scale, or should we talk about "Vertical" scaling (making a pod bigger instead of adding more)?**



To get HPA working in your local lab, we need to add one tiny but critical piece of information to your Deployment: **Resources**.

Kubernetes can't scale based on CPU usage if it doesn't know how much CPU you "expect" the app to use. It’s like a manager trying to hire more staff without knowing how much work one person can actually handle.

---

## 🛠️ Step 1: Update the Deployment

We need to tell K8s that our Python app "requests" a certain amount of CPU. Open your `lab.yaml` and update the container section:

```yaml
spec:
  containers:
  - name: python-container
    image: my-python-app:local
    resources:
      requests:
        cpu: "100m"    # 100 millicores (1/10th of a CPU core)
      limits:
        cpu: "200m"    # Don't let it go crazy and eat the whole laptop

```

Apply this change:
`kubectl apply -f lab.yaml`

---

## 🛠️ Step 2: Create the HPA

Now, let's create the "Auto-Pilot" rule. We’ll tell K8s: "If this app uses more than 50% of its requested CPU, start more pods."

Run this command:
`kubectl autoscale deployment python-lab-app --cpu-percent=50 --min=2 --max=10`

---

## 🛠️ Step 3: The "Stress Test" (Watch it Scale)

Since our app is just a "Hello World," it uses almost zero CPU. We need to **fake a heavy load** to see the HPA kick in.

1. **Open a "Watch" window:**
In a new terminal, run: `kubectl get hpa -w`
*(You'll see `TARGETS: 0%/50%`)*
2. **Generate Load:**
Run this "Infinite Loop" command in another terminal. It will hit your local service as fast as possible:
`while true; do curl -s http://localhost:30001 > /dev/null; done`
3. **Watch the Magic:**
* After about 1–2 minutes, the `TARGETS` percentage will climb (e.g., `80%/50%`).
* The `REPLICAS` column will suddenly jump from 2 to 4, then 8, then 10.
* Run `kubectl get pods` to see your new "army" of pods working to handle the traffic.



---

## 🛡️ The "Cooldown" (Scaling Back Down)

Once you stop the `while` loop (Ctrl+C), Kubernetes won't kill the pods immediately.

* **Why?** To prevent **"Flapping"** (scaling up and down every 5 seconds if traffic is "bumpy").
* **The Pro Rule:** By default, K8s waits about 5 minutes of low activity before it decides it's safe to scale back down to the `minReplicas`.

---

## 🏁 Pro Recap: HPA Best Practices

1. **Always set Resource Requests:** HPA is blind without them.
2. **Don't set thresholds too high:** If you set it to 90%, the pod might crash before the new pods have time to start. 50-70% is the "sweet spot."
3. **Check your Cluster Autoscaler:** On Google Cloud (GKE), if HPA needs 100 pods but your machines are full, the **Cluster Autoscaler** will actually go out and buy a new virtual machine from Google to fit your pods!

**You've officially automated the growth of your company! Would you like to move on to "Vertical" Pod Autoscaling (VPA)—where we make the pod stronger instead of making more of them—or are you ready for the final graduation wrap-up?**