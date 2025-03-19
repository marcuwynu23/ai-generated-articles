# Setting Up Prometheus on a VPS for Monitoring and Alerting

**Prometheus** is a powerful monitoring and alerting tool designed for cloud infrastructure, servers, and applications. If you're hosting a **VPS (Virtual Private Server)** and want to monitor metrics like **CPU, memory, disk usage, web server health**, and **custom application metrics**, this guide will walk you through setting up **Prometheus, Grafana, and Alertmanager** with **NGINX reverse proxy** for web access.

---

## 1. VPS Requirements

Before starting, ensure your VPS meets the following:

- ✅ **Ubuntu 20.04+** (or similar Linux distribution)
- ✅ **Minimum 2GB RAM** (for smooth performance)
- ✅ **Root or sudo access**
- ✅ **Firewall configured (if using UFW, allow required ports)**

---

## 2. Install Prometheus

### Step 1: Create Prometheus User

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir /etc/prometheus /var/lib/prometheus
```

### Step 2: Download and Install Prometheus

```bash
cd /opt
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-linux-amd64.tar.gz
tar xvf prometheus-linux-amd64.tar.gz
sudo mv prometheus-*/ /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/prometheus /etc/prometheus /var/lib/prometheus
```

### Step 3: Configure Prometheus

Create a configuration file at `/etc/prometheus/prometheus.yml`:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'web_app'
    static_configs:
      - targets: ['localhost:5000']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

---

## 3. Install and Configure Alertmanager

### Step 1: Download Alertmanager

```bash
cd /opt
wget https://github.com/prometheus/alertmanager/releases/latest/download/alertmanager-linux-amd64.tar.gz
tar xvf alertmanager-linux-amd64.tar.gz
sudo mv alertmanager-*/ /usr/local/bin/alertmanager
sudo mkdir /etc/alertmanager /var/lib/alertmanager
sudo chown prometheus:prometheus /usr/local/bin/alertmanager /etc/alertmanager /var/lib/alertmanager
```

### Step 2: Configure Alertmanager

Create `/etc/alertmanager/alertmanager.yml`:

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'email-notifier'

receivers:
  - name: 'email-notifier'
    email_configs:
      - to: 'your-email@example.com'
        from: 'alertmanager@example.com'
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'your-email@example.com'
        auth_password: 'your-email-password'
```

---

## 4. Install and Configure Grafana

### Step 1: Install Grafana

```bash
sudo apt install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_10.0.0_amd64.deb
sudo dpkg -i grafana_10.0.0_amd64.deb
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

### Step 2: Access Grafana

Grafana runs on **port 3000**. To access it:

- Open `http://your-vps-ip:3000/`
- Default credentials: **admin / admin**

---

## 5. Install Node Exporter (System Metrics)

```bash
cd /opt
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-linux-amd64.tar.gz
tar xvf node_exporter-linux-amd64.tar.gz
sudo mv node_exporter-*/ /usr/local/bin/node_exporter
```

Run Node Exporter:

```bash
/usr/local/bin/node_exporter &
```

---

## 6. Set Up NGINX Reverse Proxy

### Step 1: Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

### Step 2: Configure NGINX

Edit `/etc/nginx/sites-available/prometheus`:

```nginx
server {
    listen 80;
    server_name your-server-ip;

    location /prometheus/ {
        proxy_pass http://localhost:9090/;
    }

    location /grafana/ {
        proxy_pass http://localhost:3000/;
    }

    location /alertmanager/ {
        proxy_pass http://localhost:9093/;
    }
}
```

Enable the site and restart NGINX:

```bash
sudo ln -s /etc/nginx/sites-available/prometheus /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

Now you can access:

- **Prometheus** → `http://your-vps-ip/prometheus/`
- **Grafana** → `http://your-vps-ip/grafana/`
- **Alertmanager** → `http://your-vps-ip/alertmanager/`

---

## 7. Enable HTTPS with Let's Encrypt (Optional)

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.com
```

Now your services will be accessible via **HTTPS**.

---

## 8. Create an Alert for Web Server Downtime

Modify `/etc/prometheus/alerts.yml`:

```yaml
groups:
  - name: server_status_alerts
    rules:
      - alert: WebServerDown
        expr: up{job="web_app"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Web server is down"
          description: "The monitored web application has been down for more than 1 minute."
```

Reload Prometheus:

```bash
sudo systemctl restart prometheus
```

---

## 🎉 **Congratulations!**

You now have:

✅ **Prometheus for monitoring**  
✅ **Grafana for visualization**  
✅ **Alertmanager for notifications**  
✅ **NGINX for secure access**  

If you need **custom queries, dashboards, or alert integrations**, feel free to ask! 🚀
