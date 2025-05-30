# 🚀 **Guide: Implementing Staging and Production Environments in Kubernetes with Helm**

This guide will walk you through:

✅ Setting up **staging** and **production** environments
✅ Using **Helm** for deployment, updates, and rollback
✅ **Troubleshooting crashes** with `kubectl rollout restart`
✅ **Live log streaming** with Stern

---

## 🟢 **1️⃣ Environment Setup: Staging & Production**

We’ll use **Kubernetes Namespaces** to separate environments.

### Create Namespaces:

```bash
kubectl create namespace staging
kubectl create namespace production
```

✅ Namespaces ensure that:

- Staging and production workloads don’t interfere
- You can easily manage and monitor each environment separately

---

## 🟡 **2️⃣ Create Helm Chart**

Assuming you have a Helm chart for your app (e.g., `vite-app`):

```bash
helm create vite-app
```

Your chart structure:

```
vite-app/
  Chart.yaml
  values.yaml
  templates/
```

---

## 🔵 **3️⃣ Helm Install (First-time deployment)**

### Staging:

```bash
helm install vite-app ./vite-app --namespace staging \
  --set replicaCount=2 \
  --set image.tag=staging
```

### Production:

```bash
helm install vite-app ./vite-app --namespace production \
  --set replicaCount=5 \
  --set image.tag=latest
```

✅ This deploys your app with environment-specific values.

---

## 🟠 **4️⃣ Helm Upgrade (Updating your app)**

When you change the chart or image version:

```bash
helm upgrade vite-app ./vite-app --namespace staging \
  --set image.tag=staging-v2
```

For production:

```bash
helm upgrade vite-app ./vite-app --namespace production \
  --set image.tag=1.2.0
```

✅ **Tip**: Use `-f values.yaml` for more complex overrides:

```bash
helm upgrade vite-app ./vite-app --namespace staging -f values.staging.yaml
```

---

## 🟣 **5️⃣ Helm Rollback (Reverting to previous release)**

If something goes wrong:

```bash
helm rollback vite-app <REVISION> --namespace staging
```

👉 To see all revisions:

```bash
helm history vite-app --namespace staging
```

👉 Example rollback:

```bash
helm rollback vite-app 2 --namespace staging
```

---

## 🟤 **6️⃣ Live Logs: Using Stern for Real-time Monitoring**

**Stern** is an amazing tool for **live streaming logs** from multiple pods at once — color-coded and labeled!

### Install Stern:

- **macOS**: `brew install stern`
- **Windows**: `choco install stern`
- **Linux**: Download from GitHub: [https://github.com/stern/stern/releases](https://github.com/stern/stern/releases)

### Example: Live logs from all pods in `staging`

```bash
stern . -n staging
```

### Filter by app label:

```bash
stern vite-app -n staging
```

✅ Stern will:

- **Tail logs** in real-time
- Group logs by pod name
- Color-code logs for easy reading

---

## 🔴 **7️⃣ Troubleshooting CrashLoopBackOff After Rollback**

Sometimes, even after rollback, pods might enter `CrashLoopBackOff` because of:

- Bad environment variables
- ConfigMap/Secret changes
- Database issues

### Check logs:

```bash
kubectl logs -n staging <pod-name>
```

### Restart the deployment:

```bash
kubectl rollout restart deployment vite-app -n staging
```

✅ This forces Kubernetes to recreate pods cleanly with the current config.

---

## 🧠 **Key Takeaways & Best Practices**

| Task                       | Command / Tool                            |
| -------------------------- | ----------------------------------------- |
| Separate environments      | Namespaces (`staging`, `production`)      |
| Initial deployment         | `helm install`                            |
| Update application version | `helm upgrade`                            |
| Revert to stable version   | `helm rollback`                           |
| Live log streaming         | `stern`                                   |
| Troubleshoot restarts      | `kubectl logs`, `kubectl rollout restart` |
| Use staging to validate    | Deploy to `staging` before `production`   |

---

## 🚀 **Conclusion**

✅ With these tools, you’ll have:

- Controlled **staging & production environments**
- **Live log monitoring** to detect issues in real-time
- A streamlined workflow for **upgrades, rollbacks, and fixes**
