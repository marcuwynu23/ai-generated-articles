# 🛠️ How to Access Multiple Proxmox VMs with One Public IP (SSH, Web & Any TCP Service)

When running multiple VMs in a **Proxmox cluster**, you might only have **one public IP** (on Cluster A) exposed to the internet.  
If another Proxmox cluster (Cluster B) or private VMs host services, you need a way to access them remotely.

This guide explains how to expose **multiple VMs and services** (SSH, Web, Databases, RDP, etc.) behind a single public IP using **Nginx TCP stream proxy** or **iptables NAT rules**.  

---

## 📌 Scenario

- **Cluster A** → public IP (`203.0.113.10`), connected to the internet.  
- **Cluster B** → private network (`10.0.20.x`), hosts VMs.  
- Goal: Access **multiple TCP services** (SSH, Web, Databases, RDP, etc.) on Cluster B through Cluster A’s public IP.  

---

## 🔑 Approach 1: Unique External Ports for SSH (Example)

| VM Name   | Private IP    | Internal Port | Public IP (Cluster A) | External Port |
|-----------|---------------|---------------|------------------------|---------------|
| VM1       | 10.0.20.50    | 22            | 203.0.113.10           | 2221          |
| VM2       | 10.0.20.51    | 22            | 203.0.113.10           | 2222          |
| VM3       | 10.0.20.52    | 22            | 203.0.113.10           | 2223          |

```bash
ssh -p 2221 user@203.0.113.10   # VM1
ssh -p 2222 user@203.0.113.10   # VM2
ssh -p 2223 user@203.0.113.10   # VM3
```

---

### 🧩 Option A: iptables (Cluster A)
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

Reload Nginx:
```bash
nginx -s reload
```

---

## 🔑 Approach 2: Domain-Based Web Forwarding (HTTP/HTTPS)

For web servers, instead of unique ports, use **domain names**.

### Example: VM websites
- `app1.example.com` → `10.0.20.50:80`  
- `app2.example.com` → `10.0.20.51:80`  
- `app3.example.com` → `10.0.20.52:80`  

### Nginx config:
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

Enable HTTPS:
```bash
apt install -y certbot python3-certbot-nginx
certbot --nginx -d app1.example.com -d app2.example.com -d app3.example.com
```

---

## 🔑 Approach 3: Other TCP Services

This method works for **any TCP service**, not just SSH or HTTP.

### Example: Database (MySQL)
Forward MySQL VM (`10.0.20.60:3306`) → Public port `3307`:
```nginx
stream {
    server {
        listen 3307;
        proxy_pass 10.0.20.60:3306;
    }
}
```
Client connects:
```bash
mysql -h 203.0.113.10 -P 3307 -u user -p
```

### Example: Remote Desktop (RDP)
Forward RDP VM (`10.0.20.70:3389`) → Public port `3390`:
```bash
iptables -t nat -A PREROUTING -p tcp --dport 3390 -j DNAT --to-destination 10.0.20.70:3389
```

Client connects:
```bash
mstsc /v:203.0.113.10:3390
```

### Example: Custom App (port 5000)
Forward VM app (`10.0.20.80:5000`) → Public port `2050`:
```nginx
stream {
    server {
        listen 2050;
        proxy_pass 10.0.20.80:5000;
    }
}
```
Client connects:
```bash
telnet 203.0.113.10 2050
```

---

## 🔑 Approach 4: SSH Jump Host (Alternative)

Instead of assigning ports, you can use Cluster A as a **bastion/jump host**.

Add to your `~/.ssh/config`:
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

Then connect:
```bash
ssh vm1
ssh vm2
```

---

## ⚠️ Note on UDP Services

- The above works for **TCP** only.  
- For UDP services (DNS, WireGuard, VoIP), use iptables with `-p udp` or a VPN solution.  
- Example for forwarding DNS (`10.0.20.90:53`) to public port `5353`:  
```bash
iptables -t nat -A PREROUTING -p udp --dport 5353 -j DNAT --to-destination 10.0.20.90:53
```

---

## ✅ Summary

- **SSH & TCP apps**: Expose with **unique external ports** → map to internal VM IP:Port.  
- **Web apps**: Use **domain-based reverse proxy** (Nginx + Let’s Encrypt).  
- **Other TCP apps**: Same approach works (databases, RDP, custom ports).  
- **UDP apps**: Use iptables NAT or VPN.  

⚡ This setup lets you run dozens of VMs and services behind a **single public IP**, keeping access organized and secure.
