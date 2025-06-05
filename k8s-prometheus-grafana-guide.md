# 🚀 Monitoring a Kubernetes Cluster with Prometheus and Grafana

In modern cloud-native deployments, it’s essential to have **observability** into your applications and cluster. **Prometheus** and **Grafana** are the perfect combo for **monitoring Kubernetes workloads**, like your **Vite app** deployed with 3 replicas.

Here’s a **detailed, simple guide** to get everything up and running!

---

## 🟩 Step 1️⃣: Add the Prometheus Helm Repository

We’ll use **Helm** (the Kubernetes package manager) to quickly install everything.

1️⃣ **Add the Prometheus Community Helm repository**:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

2️⃣ **Update your local Helm repo**:

```bash
helm repo update
```

✅ This makes sure you have the latest charts.

---

## 🟦 Step 2️⃣: Install the Kube-Prometheus Stack

The **kube-prometheus-stack** chart bundles:

- Prometheus
- Grafana
- Node Exporter
- Kube State Metrics
- All auto-configured to monitor your Kubernetes cluster!

Install it:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack
```

or if issue with `maximumStartupDurationSeconds` remove previous installation then re-install using this:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack --set prometheus.prometheusSpec.maximumStartupDurationSeconds=300
```

- `monitoring` is the **release name**.
- This installs everything in your **current namespace** (by default, \`default\`).

---

## 🟧 Step 3️⃣: Access the Grafana UI

Grafana is the **visualization dashboard** where you’ll see metrics.

1️⃣ **Get the admin password**:

```bash
kubectl get secret --namespace default monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

2️⃣ **Forward Grafana to your local machine**:

```bash
kubectl port-forward svc/monitoring-grafana 3000:80
```

3️⃣ **Open your browser** and go to:

```
http://localhost:3000
```

- Username: \`admin\`
- Password: (the one you decoded earlier)

✅ You’re in!

---

## 🟨 Step 4️⃣: Explore Grafana Dashboards

In Grafana:

- Click on **Dashboards → Browse**.
- You’ll see **preconfigured Kubernetes dashboards**:
  - **Cluster overview**
  - **Nodes**
  - **Deployments** (like your 3-replica Vite app!)
  - **Pods**

These dashboards **automatically include metrics for your deployments**:
✅ CPU and memory usage  
✅ Pod restarts  
✅ Deployment status  
✅ Replica scaling

---

## 🟪 Step 5️⃣: Verify Your Vite App Deployment Status

Because Prometheus scrapes **Kubernetes default metrics**, you’ll see:

- How many replicas are running
- Whether any pods are in \`CrashLoopBackOff\`
- Resource usage (CPU, memory) for each Vite app pod

**No extra config needed!**

---

### 🔧 (Optional) Application-Level Metrics

If your Vite app **exports custom metrics** (e.g., on \`/metrics\` endpoint), you can configure **Prometheus** to scrape it:

1️⃣ Expose the metrics endpoint on your Vite app’s service.  
2️⃣ Create a **ServiceMonitor** or **PodMonitor** resource to tell Prometheus where to scrape.

✅ But for most cluster-level and pod-level metrics, Prometheus covers it out of the box!

---

### 🔥 Summary of Commands

Here’s a **quick cheat sheet** of what you need:

```bash

# Add Helm repo

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus + Grafana stack

helm install monitoring prometheus-community/kube-prometheus-stack

# Get Grafana password

kubectl get secret --namespace default monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Port forward Grafana to localhost:3000

kubectl port-forward svc/monitoring-grafana 3000:80
```

---

### ✅ Final Notes

🎉 **You’re done!**  
Your **Vite app** and **Kubernetes cluster** are now fully monitored with **Prometheus + Grafana**. You can:

✅ Track deployment status  
✅ Monitor pod health and resource usage  
✅ Set up alerts for failures or high load

Would you like me to **package this guide as a Markdown file** or include **bonus tips on setting up alerts in Grafana**? Let me know – I’m here to help! 🚀✨
