# How to Set Up OWASP ZAP as an API Service on a VPS

## Introduction

OWASP ZAP (Zed Attack Proxy) is one of the most powerful open-source tools for automated security scanning. If you want to deploy OWASP ZAP as a cloud-based API service to run security scans remotely, this guide will show you how to set it up on a VPS and expose it as an HTTP API.

## Step 1: Install Java (Required for ZAP)

Since OWASP ZAP requires Java, install it on your VPS first.

### For Ubuntu/Debian:

```sh
sudo apt update
sudo apt install -y default-jre
```

### For CentOS/RHEL:

```sh
sudo yum install -y java-11-openjdk
```

Verify Java installation:

```sh
java -version
```

You should see Java 11 or a newer version.

## Step 2: Download and Install OWASP ZAP

Download the latest OWASP ZAP version for Linux:

```sh
wget https://github.com/zaproxy/zaproxy/releases/download/v2.14.0/ZAP_2.14.0_Linux.tar.gz
```

Extract the files:

```sh
tar -xvzf ZAP_2.14.0_Linux.tar.gz
cd ZAP_2.14.0
```

## Step 3: Start OWASP ZAP as an API Service

Run ZAP in headless daemon mode:

```sh
./zap.sh -daemon -host 0.0.0.0 -port 8080 -config api.addrs.addr.name=*
```

This starts ZAP as a background service listening on port 8080.

## Step 4: Allow Remote API Access

To allow access from an external machine, open the firewall:

### On Ubuntu/Debian:

```sh
sudo ufw allow 8080/tcp
sudo ufw reload
```

### On CentOS/RHEL:

```sh
sudo firewall-cmd --add-port=8080/tcp --permanent
sudo firewall-cmd --reload
```

## Step 5: Create an API Endpoint for External Requests

To enable external API requests like:

```
http://owasp-zap.cloudmateria.com/scan?target=https://example.com
```

We need a Node.js Express server to act as an API gateway.

### Install Node.js

```sh
sudo apt install -y nodejs npm
```

### Set Up the API Server

```sh
mkdir zap-api
cd zap-api
npm init -y
```

Install required packages:

```sh
npm install express axios cors
```

Create `server.js`:

```js
const express = require("express");
const axios = require("axios");

const app = express();
const PORT = 3000; // API Port
const ZAP_API = "http://localhost:8080"; // Local ZAP API

app.use(express.json());

app.get("/scan", async (req, res) => {
  const target = req.query.target;
  if (!target) return res.status(400).json({ error: "Target URL is required" });
  try {
    await axios.get(`${ZAP_API}/JSON/spider/action/scan/`, {
      params: { url: target },
    });
    await axios.get(`${ZAP_API}/JSON/ascan/action/scan/`, {
      params: { url: target },
    });
    res.json({ message: "Scanning started", target });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.get("/results", async (req, res) => {
  const target = req.query.target;
  if (!target) return res.status(400).json({ error: "Target URL is required" });
  try {
    const response = await axios.get(`${ZAP_API}/JSON/core/view/alerts/`, {
      params: { baseurl: target },
    });
    res.json({ target, alerts: response.data.alerts });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(PORT, () =>
  console.log(`ZAP API Gateway running at http://localhost:${PORT}`)
);
```

## Step 6: Make the API Public with Nginx

To allow external access via `http://owasp-zap.cloudmateria.com`, configure Nginx as a reverse proxy.

### Install Nginx

```sh
sudo apt install -y nginx
```

### Configure Nginx for Reverse Proxy

Edit the default configuration:

```sh
sudo nano /etc/nginx/sites-available/default
```

Replace the content with:

```
server {
    listen 80;
    server_name owasp-zap.cloudmateria.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Save and exit.

Restart Nginx:

```sh
sudo systemctl restart nginx
```

## Step 7: Keep API Running in Background

To keep the API server running even after reboot, use PM2:

```sh
npm install -g pm2
pm2 start server.js --name zap-api
pm2 startup
pm2 save
```

## Final API Endpoints

### Start a Scan

```
GET http://owasp-zap.cloudmateria.com/scan?target=https://example.com
```

### Retrieve Scan Results

```
GET http://owasp-zap.cloudmateria.com/results?target=https://example.com
```

## Conclusion

Now, OWASP ZAP is fully deployed as an API service on your VPS, allowing external security scans via a simple HTTP request. This setup is great for automated security testing, CI/CD pipelines, and penetration testing services.

🔐 Need more security configurations? Consider adding **API authentication** (`-config api.key=YOUR_API_KEY`) and **SSL encryption with Let's Encrypt**. 🚀
