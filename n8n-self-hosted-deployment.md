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
      - N8N_HOST=n8n.cloudmateria.com
      - N8N_PROTOCOL=https
      - N8N_WEBHOOK_URL=https://n8n.cloudmateria.com
      - N8N_EDITOR_BASE_URL=https://n8n.cloudmateria.com
      - N8N_WEBHOOK_TUNNEL_URL=https://n8n.cloudmateria.com
      - N8N_PUBLIC_API_DISABLED=false
      - N8N_TRUST_PROXY=true
      - N8N_TRUSTED_PROXIES=127.0.0.1,::1
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
      - WEBHOOK_URL=https://n8n.cloudmateria.com
    volumes:
      - data:/home/node/.n8n
    ports:
      - "5678:5678"
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
# Redirect all HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name n8n.cloudmateria.com;

    return 301 https://$host$request_uri;
}

# HTTPS reverse proxy for n8n
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name n8n.cloudmateria.com;

    ssl_certificate     /etc/nginx/ssl/cloudflare.pem;
    ssl_certificate_key /etc/nginx/ssl/cloudflare.key;

    client_max_body_size 100M;

    # 🚫 Prevent browsers from attempting QUIC/HTTP3
    add_header Alt-Svc "" always;
    add_header QUIC-Status "disabled" always;

    # Main n8n editor and API
    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        proxy_buffering off;
        proxy_request_buffering off;
        proxy_cache_bypass $http_upgrade;
    }

    location /webhook/ {
        proxy_pass http://127.0.0.1:5678;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        # proxy_buffering off;
        # proxy_request_buffering off;
        # proxy_cache_bypass $http_upgrade;
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
