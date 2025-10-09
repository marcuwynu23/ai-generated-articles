# 🎧 Secure SIP + RTP over WireGuard

_(Drachtio Server + Java Clients)_

---

## 🧩 Overview

This guide describes how to securely tunnel Drachtio SIP signaling and RTP audio through a WireGuard VPN.  
It ensures encrypted, NAT-proof communication between your SIP server and Java clients.

---

## 🏗️ Network Plan

| Role                 | Real IP         | WireGuard IP | Description   |
| -------------------- | --------------- | ------------ | ------------- |
| Drachtio Server      | `172.13.62.203` | `10.66.0.1`  | SIP + RTP Hub |
| Node 1 (Java Client) | (Home PC)       | `10.66.0.2`  | First caller  |
| Node 2 (Java Client) | (Laptop)        | `10.66.0.3`  | Second caller |

---

## ⚙️ 1 · Install WireGuard

### On Ubuntu / Debian

```bash
sudo apt update
sudo apt install wireguard -y
```

### On Windows clients

1. Download from <https://www.wireguard.com/install/>
2. Click **Add Tunnel → Add empty tunnel**
3. Paste the client configuration (see below).

---

## 🗝️ 2 · Generate keys

### Server

```bash
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

### Each Client

```bash
wg genkey | tee client_private.key | wg pubkey > client_public.key
```

Keep all keys safe.

---

## 🧱 3 · Configure Server (`/etc/wireguard/wg0.conf`)

```ini
[Interface]
Address = 10.66.0.1/24
ListenPort = 51820
PrivateKey = <SERVER_PRIVATE_KEY>

# Client 1
[Peer]
PublicKey = <CLIENT1_PUBLIC_KEY>
AllowedIPs = 10.66.0.2/32

# Client 2
[Peer]
PublicKey = <CLIENT2_PUBLIC_KEY>
AllowedIPs = 10.66.0.3/32
```

Enable IP forwarding:

```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Start WireGuard:

```bash
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```

---

## 💻 4 · Configure Clients

### Client 1 (`/etc/wireguard/wg0.conf`)

```ini
[Interface]
Address = 10.66.0.2/24
PrivateKey = <CLIENT1_PRIVATE_KEY>
DNS = 1.1.1.1

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = 172.13.62.203:51820
AllowedIPs = 10.66.0.0/24
PersistentKeepalive = 25
```

### Client 2

```ini
[Interface]
Address = 10.66.0.3/24
PrivateKey = <CLIENT2_PRIVATE_KEY>

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = 172.13.62.203:51820
AllowedIPs = 10.66.0.0/24
PersistentKeepalive = 25
```

Activate:

```bash
sudo wg-quick up wg0
```

Verify:

```bash
ping 10.66.0.1
ping 10.66.0.2
ping 10.66.0.3
```

---

## 📦 5 · Configure Drachtio Server

Edit `/etc/drachtio/drachtio.conf.xml`:

```xml
<drachtio>
  <admin port="9022" secret="c0mpl3x">127.0.0.1</admin>
  <sip>
    <contacts>
      <contact>sip:10.66.0.1:5070;transport=udp,tcp</contact>
    </contacts>
  </sip>
</drachtio>
```

Restart Drachtio:

```bash
sudo systemctl restart drachtio
```

---

## 🎙️ 6 · Configure Java Clients

Edit in `SimpleSipRtpClient.java`:

```java
private static String SIP_SERVER = "10.66.0.1"; // WireGuard IP of Drachtio
```

Compile and run:

```bash
javac SimpleSipRtpClient.java
java SimpleSipRtpClient node1 node2 10.66.0.1 5070
java SimpleSipRtpClient node2 node1 10.66.0.1 5070
```

Both clients will exchange RTP seamlessly inside the WireGuard VPN.

---

## 🔐 7 · Firewall Rules

```bash
sudo ufw allow 51820/udp    # WireGuard
sudo ufw allow 5070/udp     # SIP
sudo ufw allow 4000:5000/udp  # RTP (optional if tunneled only)
```

---

## 🧪 8 · Verify Tunnel and RTP

Check WireGuard status:

```bash
sudo wg show
```

Example Java console log:

```
[node1] RTP packet from 10.66.0.3:4750 (372 bytes)
[node1] Echoed audio back to 10.66.0.3:4750
```

---

## 🧠 Tips

- WireGuard eliminates NAT traversal problems.
- All SIP/RTP traffic is encrypted (ChaCha20).
- You can add multiple LGUs or barangay nodes by appending more `[Peer]` blocks.

---

## 🧾 Summary

| Component | IP            | Role                |
| --------- | ------------- | ------------------- |
| Drachtio  | 10.66.0.1     | SIP Server          |
| Node 1    | 10.66.0.2     | Java Client A       |
| Node 2    | 10.66.0.3     | Java Client B       |
| Transport | WireGuard VPN | Encrypted SIP + RTP |
