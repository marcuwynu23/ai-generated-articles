# How to Set Up VS Code in the Browser with Code Server

Running **VS Code in the browser** is a great way to access your development environment from anywhere. This guide will walk you through setting up **Code Server**, configuring it as a service, securing it with **NGINX and SSL**, and enabling authentication for protection.

---

## **1. Install Code Server**

First, install **Code Server** on your Linux server or local machine.

```sh
curl -fsSL https://code-server.dev/install.sh | sh
```

After installation, start Code Server manually:

```sh
code-server
```

By default, it runs on `http://localhost:8080` with a password stored in:

```
/root/.config/code-server/config.yaml
```

Find your password with:

```sh
grep password /root/.config/code-server/config.yaml
```

To change it, edit the config file:

```sh
sudo nano /root/.config/code-server/config.yaml
```

Modify the `password` field and save the file.

---

## **2. Set Up Code Server as a Systemd Service**

To ensure that **Code Server starts automatically**, create a systemd service file:

```sh
sudo nano /etc/systemd/system/code-server.service
```

Add the following content:

```ini
[Unit]
Description=Code Server
After=network.target

[Service]
Type=simple
User=your-username
WorkingDirectory=/home/your-username
ExecStart=/usr/bin/code-server --bind-addr 0.0.0.0:8080 /path/to/your/directory
Restart=always
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

Replace:

- `your-username` with your Linux username.
- `/path/to/your/directory` with the folder you want to work in.

Save and close the file, then reload systemd:

```sh
sudo systemctl daemon-reload
sudo systemctl enable code-server
sudo systemctl start code-server
```

To check if it's running:

```sh
sudo systemctl status code-server
```

---

## **3. Configure NGINX as a Reverse Proxy**

To make Code Server accessible via a domain and secure it with SSL, set up an **NGINX reverse proxy**.

### **3.1 Install NGINX**

```sh
sudo apt update
sudo apt install nginx -y
```

### **3.2 Create a Reverse Proxy Configuration**

```sh
sudo nano /etc/nginx/sites-available/code-server
```

Add the following:

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
```

Replace `your-domain.com` with your actual domain.

### **3.3 Enable the Configuration**

```sh
sudo ln -s /etc/nginx/sites-available/code-server /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## **4. Secure Code Server with SSL (HTTPS)**

To secure Code Server, install **Certbot** and obtain a free SSL certificate:

```sh
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.com
```

Certbot will automatically update your NGINX configuration to use HTTPS.

To ensure SSL is automatically renewed:

```sh
sudo crontab -e
```

Add:

```
0 0 * * * certbot renew --quiet && systemctl restart nginx
```

---

## **5. Enable Basic Authentication Using Code Server Password**

To use the **existing Code Server password** for authentication, modify the NGINX config:

```sh
sudo nano /etc/nginx/sites-available/code-server
```

Update the `location /` block as follows:

```nginx
location / {
    auth_request /auth;
    proxy_pass http://localhost:8080/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

location = /auth {
    internal;
    proxy_pass http://localhost:8080/auth;
}
```

Then restart NGINX:

```sh
sudo systemctl restart nginx
```

---

## **6. Access VS Code in the Browser**

Now, open **https://your-domain.com** in a browser and enter the **Code Server password** from `/root/.config/code-server/config.yaml`.

---

### ✅ **Now, your VS Code Web is running securely with authentication and SSL!** 🚀

Would you like to restrict access to certain IPs for additional security? 🔒
