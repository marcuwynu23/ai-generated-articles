
# 🌐 Exposing Multiple Kubernetes Apps via a Single NGINX Reverse Proxy and Cloudflare Tunnel

Running Kubernetes inside a Proxmox VM? Want to securely expose multiple apps (`app1`, `app2`, `service1`, etc.) to the internet **using one reverse proxy** and **Cloudflare Tunnel**?

This article walks you through building a **production-ready, scalable setup** that maps:

```
https://app1.cloudmateria.com → app1-service in K8s
https://service3.cloudmateria.com → service3-service in K8s
```

All through:

- ✅ Kubernetes Ingress Controller
- ✅ NGINX on the Proxmox host
- ✅ Cloudflared Tunnel
- ✅ *.cloudmateria.com wildcard DNS

---

## ✅ Architecture Overview

```
Internet (app1.cloudmateria.com)
↓
Cloudflare Tunnel
↓
NGINX (Proxmox Host, port 80)
↓
Kubernetes Ingress Controller (NodePort)
↓
K8s Ingress Rule → Service → Pod
```

---

## 🧱 Step 1: Deploy Apps in Kubernetes

Each app should have:

### 🧩 Deployment (`app1-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
        - name: app1
          image: your/image:tag
          ports:
            - containerPort: 3000
```

### 🧩 Service (`app1-service.yaml`)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-service
spec:
  selector:
    app: app1
  ports:
    - port: 80
      targetPort: 3000
```

### 🧩 Ingress (`app1-ingress.yaml`)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app1-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: app1.cloudmateria.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 80
```

📦 Repeat for each app: `app2`, `app3`, `service1`, etc.

---

## ⚙️ Step 2: Install Ingress Controller in Kubernetes VM

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```

Then confirm the NodePort:

```bash
kubectl get svc -n ingress-nginx
```

You should see something like:

```
ingress-nginx-controller   NodePort   10.43.x.x   <none>   80:30702/TCP
```

---

## 🌐 Step 3: Wildcard Reverse Proxy via NGINX on Proxmox

### 📄 `/etc/nginx/sites-available/cloudmateria.com`
```nginx
server {
  listen 80 default_server;
  server_name ~^(?<subdomain>.+)\.cloudmateria\.com$;

  location / {
    proxy_pass http://192.168.1.101:30702; # IP of your Kubernetes VM
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```

Enable the site:

```bash
ln -s /etc/nginx/sites-available/cloudmateria.com /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

---

## ☁️ Step 4: Cloudflared Tunnel Setup

### 📄 `/etc/cloudflared/config.yml`
```yaml
tunnel: my-k8s-tunnel
credentials-file: /root/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: "*.cloudmateria.com"
    service: http://localhost:80

  - service: http_status:404
```

Restart Cloudflared:

```bash
systemctl restart cloudflared
```

---

## 🌍 Step 5: Cloudflare DNS Configuration

Add a wildcard DNS record in your Cloudflare dashboard:

| Type | Name | Target |
|------|------|--------|
| CNAME | `*.cloudmateria.com` | `<your-tunnel-id>.cfargotunnel.com` |

✅ Enable proxy (orange cloud) to route through Cloudflare.

---

## ✅ Final Result

Now all these will work:

| Domain | Backend |
|--------|---------|
| `app1.cloudmateria.com` | `app1-service` in Kubernetes |
| `service3.cloudmateria.com` | `service3-service` in Kubernetes |
| `app2.cloudmateria.com` | `app2-service`, and so on |

💨 All traffic flows through Cloudflare Tunnel → NGINX on Proxmox → K8s Ingress → your services.

---

## 🔐 Bonus: Security & Extras

- Add **Cloudflare Access** for authentication
- Use `cert-manager` inside K8s for internal TLS
- Add `rate limiting` or `auth` via NGINX

---

## ✅ Conclusion

You now have a modern reverse proxy setup with:

- ✅ **Wildcard domain routing**
- ✅ **Single entry point (NGINX + Cloudflared)**
- ✅ **Scalable Ingress-based Kubernetes backend**

This pattern is perfect for managing **multiple microservices**, admin tools, or internal dashboards.
