# How WireGuard VPN Works and Why Client Internet May Not Work

WireGuard is a modern, lightweight, and highly secure VPN solution. While it is simple to set up, understanding how it routes traffic is crucial to ensure clients can access both the VPN and the internet.

---

## 1. How WireGuard Works

WireGuard operates at **layer 3 (IP level)** and uses **peer-to-peer encryption** to securely route packets between devices.

### Key Components

1. **Keys:** Each device has a private key and a public key. Private keys remain secret, public keys are shared with peers.
2. **Virtual Network Interface:** Each peer has a virtual interface (`wg0`) with a VPN IP (e.g., `10.0.0.1` for the server, `10.0.0.2` for a client).
3. **AllowedIPs:** Specifies which IPs should be routed through the tunnel. This acts as both routing and access control.
4. **PersistentKeepalive:** Keeps connections alive if a peer is behind NAT.

### Traffic Flow
- Traffic destined to IPs in `AllowedIPs` is encrypted and sent through the tunnel.
- Traffic not listed in `AllowedIPs` is routed normally through the local network.

---

## 2. Why Clients Can Connect But Have No Internet

WireGuard only routes traffic specified in `AllowedIPs`. Common causes of internet issues:

### Case 1 — Split Tunnel (Default)
- Server allows traffic only to VPN IPs:
  ```ini
  [Peer]
  AllowedIPs = 10.0.0.2/32
  ```
- Only peer-to-peer traffic works. Internet traffic still uses the client’s local network.

### Case 2 — Full Tunnel But NAT Missing
- Client routes all traffic through VPN (`AllowedIPs = 0.0.0.0/0`), but server does not perform NAT.
- Packets reach the server but cannot reach the internet because the source IP is the private VPN IP.
- **Fix:** Enable NAT on server:
  ```bash
  iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  ```
  Replace `eth0` with your server’s public interface.

### Case 3 — Firewall Blocks Forwarding
- Even with NAT, firewall may block traffic from `wg0` to `eth0`.
- **Fix:**
  ```bash
  iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT
  iptables -A FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
  ```

---

## 3. Peer-to-Peer Access

By default, WireGuard allows **only traffic defined in `AllowedIPs`**:
- Server and clients can communicate if AllowedIPs are configured correctly.
- Internet access requires NAT and appropriate AllowedIPs.
- Multiple clients require server `[Peer]` entries for each VPN IP.

---

## 4. Summary Table

| Feature | Default Behavior | Solution if Not Working |
|---------|-----------------|------------------------|
| Client → Server | Works if AllowedIPs match | Already works |
| Client → Internet | Does not work | Enable full tunnel (`AllowedIPs=0.0.0.0/0`) + NAT on server |
| Client → Other Peers | Works if AllowedIPs match | Ensure all peers’ AllowedIPs include VPN subnet |
| NAT | Not automatic | Use iptables MASQUERADE on server |
| Firewall | Blocks unknown traffic | Allow UDP 51820 and forward wg0 → eth0 |

---

**Key Takeaway:** WireGuard only routes traffic you explicitly define. Peer-to-peer access works by default if AllowedIPs match, but internet routing requires server NAT and correct AllowedIPs on the client.

---

WireGuard is simple, fast, and secure—but understanding routing and NAT is essential for full functionality.

