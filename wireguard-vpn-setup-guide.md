# WireGuard VPN Installation & Setup Guide

WireGuard is a modern, fast, and secure VPN solution that is easy to set up and configure. This guide will walk you through installing WireGuard on a Linux server and configuring a Windows/Linux client to connect, including routing internet traffic through the VPN.

---

## 1. Prerequisites

- A Linux server with root access (Ubuntu/Debian/CentOS).
- A client machine (Windows, Linux, or macOS) with WireGuard installed.
- Basic knowledge of network interfaces and IP addresses.
- Firewall access to allow UDP traffic on port `51820`.

---

## 2. Install WireGuard

### Ubuntu/Debian
```bash
sudo apt update
sudo apt install wireguard -y
```

### CentOS/RHEL
```bash
sudo yum install epel-release -y
sudo yum install wireguard-tools -y
```

Check installation:
```bash
wg --version
```

---

## 3. Generate Server Keys

Generate a private and public key pair on the server:
```bash
cd /etc/wireguard
umask 077
wg genkey | tee server_private.key | wg pubkey > server_public.key
```

- `server_private.key` → used in the server config.
- `server_public.key` → provided to clients as the server’s public key.

---

## 4. Server Configuration

Create `/etc/wireguard/wg0.conf`:

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = <Server_Private_Key>
DNS = 8.8.8.8

PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <Client_Public_Key>
AllowedIPs = 10.0.0.2/32
```

> Replace `eth0` with your server’s primary public-facing interface (e.g., `ens3`).

---

## 5. Firewall Configuration

Allow UDP port `51820` through the firewall:

```bash
# UFW
sudo ufw allow 51820/udp
sudo ufw reload

# Or iptables
sudo iptables -A INPUT -p udp --dport 51820 -j ACCEPT
```

Enable IP forwarding permanently:
```bash
sudo sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sudo sysctl -p
```

---

## 6. Start WireGuard Server

```bash
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0
```

Check status:
```bash
sudo wg show
```

---

## 7. Client Configuration

Example client config (Windows/Linux):

```ini
[Interface]
Address = 10.0.0.2/32
PrivateKey = <Client_Private_Key>
# PersistentKeepalive = 25 (optional)

[Peer]
PublicKey = <Server_Public_Key>
Endpoint = <Server_Public_IP>:51820
AllowedIPs = 0.0.0.0/0
DNS = 10.0.0.1
```

---

## 8. Start WireGuard Client

- **Windows:** Import config into WireGuard app → Activate tunnel.  
- **Linux:** Place config in `/etc/wireguard/wg0.conf` → `sudo wg-quick up wg0`.

Check handshake:
```bash
wg show
```

---

## 9. Test Connection

- Ping server from client: `ping 10.0.0.1`
- Ping client from server: `ping 10.0.0.2`
- Test internet routing (full tunnel): `curl ifconfig.me` → should show server’s public IP.

---

## 10. Optional: SSH Access Between Peers

- **Server → Client:** `ssh user@10.0.0.2`
- **Client → Server:** `ssh root@10.0.0.1`

Ensure `sshd` is running and firewall allows port 22 on the WireGuard interface.

---

## 11. Notes

- **Full Tunnel vs Split Tunnel:**
  - Full tunnel (`AllowedIPs = 0.0.0.0/0`) routes all traffic through VPN (requires server NAT).
  - Split tunnel (`AllowedIPs = 10.0.0.0/24`) routes only VPN subnet traffic; client internet stays local.
- **PersistentKeepalive:** Useful for clients behind NAT.
- **Multiple clients:** Repeat `[Peer]` sections on the server for each client.

---

✅ This guide provides a complete, working setup for a secure WireGuard VPN, enabling clients to connect, access internal resources, and optionally route internet traffic through the server.

