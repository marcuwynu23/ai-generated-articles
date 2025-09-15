# 🛠️ How to Access Multiple Proxmox VMs with One Public IP (SSH & Web)

When running multiple VMs in a **Proxmox cluster**, you might only have **one public IP** (on Cluster A) exposed to the internet. If another Proxmox cluster (Cluster B) or private VMs host services, you need a way to access them remotely.

This guide explains how to expose **multiple VMs** behind a single public IP using **Nginx TCP stream proxy** or **iptables NAT rules**.  

---

## 📌 Scenario

- **Cluster A** → public IP (`203.0.113.10`), connected to the internet.  
- **Cluster B** → private network (`10.0.20.x`), hosts VMs.  
- Goal: Access **multiple SSH servers** (port 22) and **web servers** (ports 80/443) on Cluster B through Cluster A’s public IP.  

---

## 🔑 Approach 1: Unique External Ports for SSH

Since only **one IP** is available, we map **different external ports** to each VM’s internal SSH port.

| VM Name   | Private IP    | Internal Port | Public IP (Cluster A) | External Port |
|-----------|---------------|---------------|------------------------|---------------|
| VM1       | 10.0.20.50    | 22            | 203.0.113.10           | 2221          |
| VM2       | 10.0.20.51    | 22            | 203.0.113.10           | 2222          |
| VM3       | 10.0.20.52    | 22            | 203.0.113.10           | 2223          |

Now you can connect like this:
```bash
ssh -p 2221 user@203.0.113.10   # VM1
ssh -p 2222 user@203.0.113.10   # VM2
ssh -p 2223 user@203.0.113.10   # VM3
```

---

### 🧩 Option A: iptables (Cluster A)
On **Cluster A**, run:
```bash
iptables -t nat -A PREROUTING -p tcp --dport 2221 -j DNAT --to-destination 10.0.20.50:22
iptables -t nat -A PREROUTING -p tcp --dport 2222 -j DNAT --to-destination 10.0.20.51:22
iptables -t nat -A PREROUTING -p tcp --dport 2223 -j DNAT --to-destination 10.0.20.52:22
```

Save the rules:
```bash
iptables-save > /etc/iptables/rules.v4
```

---

### 🧩 Option B: Nginx TCP Stream Proxy
Nginx can forward raw TCP traffic (not just HTTP).  

1. Install Nginx (with stream module):
```bash
apt update && apt install -y nginx
```

2. Add config (e.g., `/etc/nginx/conf.d/ssh-proxy.conf`):
```nginx
stream {
    server {
        listen 2221;
        proxy_pass 10.0.20.50:22;
    }
    server {
        listen 2222;
        proxy_pass 10.0.20.51:22;
    }
    server {
        listen 2223;
        proxy_pass 10.0.20.52:22;
    }
}
```

3. Reload Nginx:
```bash
nginx -s reload
```

Now Cluster A forwards SSH ports to different VMs.

---

## 🔑 Approach 2: Domain-Based Web Forwarding

For web services (HTTP/HTTPS), you can forward by **domain name** instead of ports.

### Example: VM websites
- `app1.example.com` → `10.0.20.50:80`  
- `app2.example.com` → `10.0.20.51:80`  
- `app3.example.com` → `10.0.20.52:80`  

### Nginx config (`/etc/nginx/sites-available/web-proxy.conf`):
```nginx
server {
    listen 80;
    server_name app1.example.com;
    location / {
        proxy_pass http://10.0.20.50:80;
    }
}

server {
    listen 80;
    server_name app2.example.com;
    location / {
        proxy_pass http://10.0.20.51:80;
    }
}

server {
    listen 80;
    server_name app3.example.com;
    location / {
        proxy_pass http://10.0.20.52:80;
    }
}
```

Enable HTTPS with Let’s Encrypt:
```bash
apt install -y certbot python3-certbot-nginx
certbot --nginx -d app1.example.com -d app2.example.com -d app3.example.com
```

Now each VM has its own secure domain.

---

## 🔑 Approach 3: SSH Jump Host (Alternative)

Instead of managing ports, you can use **Cluster A as a jump host**.

Add to your client `~/.ssh/config`:
```ssh-config
Host clusterA
    HostName 203.0.113.10
    User admin

Host vm1
    HostName 10.0.20.50
    User user
    ProxyJump clusterA

Host vm2
    HostName 10.0.20.51
    User user
    ProxyJump clusterA
```

Then just connect with:
```bash
ssh vm1
ssh vm2
```

---

## ✅ Summary

- For **SSH**: Use **unique ports per VM** (via iptables or Nginx `stream`) OR use **jump host**.  
- For **Web**: Use **domain-based reverse proxy** (Nginx + Let’s Encrypt).  
- Only one **public IP** is required — Cluster A acts as a **gateway** to Cluster B VMs.  

⚡ This setup lets you run dozens of private VMs behind **one IP** while keeping access clean and secure.
