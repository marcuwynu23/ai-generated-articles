---
title: Drachtio SIP + RTP Server Setup Guide
description: Step-by-step guide to build, configure, and run Drachtio SIP signaling server with RTP relay integration on Ubuntu or Debian.
lastUpdated: 2025-10-09
---

# 🛰️ Drachtio SIP + RTP Server Setup Guide (2025)

## 🧩 Overview

**Drachtio** is an open-source SIP signaling server that enables developers to control SIP call flow, routing, and communication logic directly from Node.js or other backend applications.

This guide covers:
- Building **Drachtio** from source (latest version)
- Configuring **SIP (UDP/TCP)** and **Admin API (TCP 9022)**
- Enabling **Prometheus monitoring** and **logging**
- Connecting your **Node.js SIP+RTP relay app**

---

## ⚙️ Prerequisites

| Requirement | Description |
|--------------|-------------|
| OS | Ubuntu 20.04+, Debian 11+ |
| Privileges | Root or sudo access |
| Network | SIP (UDP/TCP 5070), Admin (TCP 9022) |
| Packages | build-essential, libboost, libtool, libssl, cmake, git |

---

## 🧱 Step 1. Install Dependencies

```bash
sudo apt update
sudo apt install -y \
  build-essential automake autoconf git cmake \
  libboost-all-dev libssl-dev libcurl4-openssl-dev \
  libpq-dev libexpat1-dev pkg-config libtool libtool-bin
```

---

## 📦 Step 2. Clone and Prepare the Source

```bash
cd ~
git clone https://github.com/davehorton/drachtio-server.git
cd drachtio-server
git submodule update --init --recursive
```

---

## 🏗️ Step 3. Build and Install

```bash
./bootstrap.sh
./configure
make -j$(nproc)
sudo make install
```

If you encounter a `libtool` error:
```bash
sudo apt install -y libtool libtool-bin
sudo make install
```

Verify installation:
```bash
which drachtio
drachtio -v
```

Expected:
```
/usr/local/bin/drachtio
Starting drachtio version v0.9.3
```

---

## ⚙️ Step 4. Configuration File

Create the config directory and file:

```bash
sudo mkdir -p /etc/drachtio
sudo nano /etc/drachtio/drachtio.conf.xml
```

Paste this configuration:

```xml
<drachtio>

  <!-- Admin TCP socket for Node.js relay control -->
  <admin port="9022" secret="c0mpl3x">127.0.0.1</admin>

  <!-- SIP stack configuration -->
  <sip>
    <contacts>
      <!-- Listen on both UDP and TCP, port 5070 -->
      <contact>sip:172.13.62.203:5070;transport=udp,tcp</contact>
    </contacts>

    <!-- Optional: block obvious scanners -->
    <spammers action="reject" tcp-action="discard">
      <header name="User-Agent">
        <value>sip-cli</value>
        <value>sipcli</value>
        <value>friendly-scanner</value>
      </header>
      <header name="To">
        <value>sipvicious</value>
      </header>
    </spammers>
  </sip>

  <!-- Metrics for Prometheus (optional) -->
  <monitoring>
    <prometheus port="8088">127.0.0.1</prometheus>
  </monitoring>

  <!-- Logging configuration -->
  <logging>
    <file>
      <name>/var/log/drachtio/drachtio.log</name>
      <archive>/var/log/drachtio/archive</archive>
      <size>100</size>
      <maxFiles>20</maxFiles>
      <auto-flush>true</auto-flush>
    </file>
    <sofia-loglevel>3</sofia-loglevel>
    <loglevel>info</loglevel>
  </logging>

</drachtio>
```

---

## 🧰 Step 5. Systemd Service

```bash
sudo nano /etc/systemd/system/drachtio.service
```

Paste:

```ini
[Unit]
Description=Drachtio SIP Server
After=network.target

[Service]
ExecStart=/usr/local/bin/drachtio -f /etc/drachtio/drachtio.conf.xml
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable drachtio
sudo systemctl start drachtio
sudo systemctl status drachtio
```

Expected:
```
● drachtio.service - Drachtio SIP Server
     Active: active (running)
```

---

## 🌐 Step 6. Open Firewall Ports

```bash
sudo ufw allow 5070/udp
sudo ufw allow 5070/tcp
sudo ufw allow 9022/tcp
sudo ufw reload
```

---

## 🧪 Step 7. Verify

```bash
sudo ss -tulnp | egrep '5070|9022|8088'
```

Expected:
```
udp   UNCONN 0 0 172.13.62.203:5070  0.0.0.0:* users:(("drachtio",pid=3120607))
tcp   LISTEN 0 4096 127.0.0.1:9022   0.0.0.0:* users:(("drachtio",pid=3120607))
tcp   LISTEN 0 4096 127.0.0.1:8088   0.0.0.0:* users:(("drachtio",pid=3120607))
```

---

## 💬 Step 8. Connect from Node.js

Install:
```bash
npm install drachtio-client
```

Example client:
```js
import { DrachtioClient } from 'drachtio-client';

const app = new DrachtioClient();

app.connect({
  host: '127.0.0.1',
  port: 9022,
  secret: 'c0mpl3x'
});

app.on('connect', () => console.log('✅ Connected to Drachtio'));
```

---

## 📈 Step 9. Monitor & Logs

Metrics (Prometheus):
```bash
curl http://127.0.0.1:8088/metrics
```

Logs:
```bash
sudo tail -f /var/log/drachtio/drachtio.log
```

---

## ✅ Summary

| Component | Port | Description |
|------------|------|-------------|
| SIP | 5070 (UDP/TCP) | Public signaling |
| Admin API | 9022 (TCP) | Local Node.js control |
| Prometheus | 8088 | Metrics endpoint |

You now have a **fully working Drachtio SIP + RTP signaling server**, ready to pair with your **Node.js relay system** for full-duplex audio communication.

---
