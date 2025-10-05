# Hosting n8n on a VPS with Docker, systemd, and HTTPS

Workflow automation tools like **n8n** are powerful, but you don’t want to run them carelessly exposed to the internet. In this article, we’ll walk through deploying n8n on a VPS with Docker, managing it as a `systemctl` service, and securing it with HTTPS using NGINX and Let’s Encrypt.

---

## 1. Prepare the VPS

Start with a Linux VPS (Ubuntu 22.04 works well). Make sure you have:

- Docker and Docker Compose installed.
- A domain name pointing to your VPS IP.

---

## 2. Docker Compose for n8n

Create a working directory:

```bash
sudo mkdir -p /opt/n8n
cd /opt/n8n
```

Create a `docker-compose.yml`:

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    container_name: n8n
    restart: unless-stopped
    network_mode: host
    environment:
      - TZ=Asia/Manila
      - DB_SQLITE_POOL_SIZE=5
      - N8N_RUNNERS_ENABLED=true
      - N8N_BLOCK_ENV_ACCESS_IN_NODE=false
      - N8N_GIT_NODE_DISABLE_BARE_REPOS=true
      - N8N_COMMUNITY_NODES_ENABLED=false
      - N8N_DIAGNOSTICS_ENABLED=false
      - N8N_TELEMETRY_ENABLED=false
      - N8N_DEFAULT_TIMEOUT=120000
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=strongpassword
    volumes:
      - data:/home/node/.n8n

volumes:
  data:
    driver: local
```

Bring it up once to check:

```bash
docker compose up -d
```

Visit `http://<your-vps-ip>:5678` to confirm n8n runs.

---

## 3. Run n8n as a systemd service

Instead of manually starting n8n, create a service unit file.

Create `/etc/systemd/system/n8n.service`:

```ini
[Unit]
Description=n8n workflow automation
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/n8n
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

Enable and start it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable n8n
sudo systemctl start n8n
```

Now n8n runs on boot and can be managed with:

```bash
sudo systemctl status n8n
sudo systemctl restart n8n
```

---

## 4. Secure with NGINX and HTTPS

You don’t want to expose n8n raw over HTTP. Instead, put it behind an NGINX reverse proxy and add a TLS certificate.

Install NGINX and Certbot:

```bash
sudo apt install nginx certbot python3-certbot-nginx -y
```

Create a config `/etc/nginx/sites-available/n8n.conf`:

```nginx
server {
    listen 80;
    server_name n8n.example.com;

    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
    }
}
```

Enable it:

```bash
sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Now request an SSL certificate:

```bash
sudo certbot --nginx -d n8n.example.com
```

Certbot will edit the config to include HTTPS. You can now access:

```
https://n8n.example.com
```

---

## 5. Automatic Renewal

Let’s Encrypt certs expire every 90 days. Certbot installs a systemd timer that handles renewal. You can test it:

```bash
sudo certbot renew --dry-run
```

---

## Conclusion

You now have:

- **n8n running in Docker** for easy management.
- **systemd integration** for reliable auto-start.
- **NGINX reverse proxy + HTTPS** for secure access.

This setup makes your VPS-hosted n8n production-ready, and you can safely begin building automations accessible from anywhere.
