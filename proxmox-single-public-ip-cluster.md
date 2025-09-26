# 🌐 How to Use One Static Public IP with Two Proxmox VE Servers

Running a **Proxmox VE cluster** with multiple nodes is common, but what if you only purchased **one static public IP address** from your data center or ISP? Don’t worry — you can still manage your cluster, run VMs, and expose services to the internet. The trick is to **choose one node as the public gateway** and use **NAT with nftables** to forward traffic into your private subnet.

---

## 🔑 The Challenge

- You have **two servers**: `pve1` and `pve2`.
- You only own **one public IP** (example: `203.0.113.25`).
- Both servers need to run VMs, but only one IP can be exposed to the internet.

**Solution:**  
👉 Assign the public IP to one node (gateway node).  
👉 Keep both nodes in a private subnet (e.g., `192.168.100.0/24`).  
👉 Use `nftables` on the gateway node to port-forward traffic to VMs on either node.

---

## 🏗️ Network Architecture

```
                 Internet
                     │
          Public IP (203.0.113.25)
                     │
            ┌────────┴─────────┐
            │  Gateway Node    │  (pve1 or pve2)
            │  - Holds public IP
            │  - Runs nftables
            └────────┬─────────┘
                     │ vmbr0 (192.168.100.0/24)
            ┌────────┴─────────┐
          pve1               pve2
      (VMs inside)       (VMs inside)
```

---

## ⚙️ Step 1. Configure Networking

### On Gateway Node (example: `pve1`)

`/etc/network/interfaces`:

```ini
auto lo
iface lo inet loopback

# NIC connected to your data center uplink
iface eno1 inet manual

# Bridge with both Public and Private IPs
auto vmbr0
iface vmbr0 inet static
    address 203.0.113.25/24   # Public IP
    gateway 203.0.113.1       # ISP Gateway
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0

iface vmbr0 inet static
    address 192.168.100.1/24  # Private LAN for VMs
```

### On Second Node (example: `pve2`)

`/etc/network/interfaces`:

```ini
auto lo
iface lo inet loopback

iface eno1 inet manual

auto vmbr0
iface vmbr0 inet static
    address 192.168.100.2/24
    bridge-ports eno1
    bridge-stp off
    bridge-fd 0
```

---

## ⚙️ Step 2. Install & Enable nftables

On the gateway node:

```bash
apt update
apt install nftables -y
systemctl enable nftables
systemctl start nftables
```

---

## ⚙️ Step 3. Configure NAT & Port Forwarding

Edit `/etc/nftables.conf` on the gateway node:

```nft
#!/usr/sbin/nft -f
flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0;
        policy drop;

        # allow localhost
        iif lo accept

        # allow established/related
        ct state established,related accept

        # allow ssh + proxmox web
        tcp dport {22, 8006} accept

        # allow ping
        ip protocol icmp accept
        ip6 nexthdr icmpv6 accept
    }

    chain forward {
        type filter hook forward priority 0;
        policy drop;

        # allow established/related
        ct state established,related accept

        # allow VM subnet forwarding
        iif vmbr0 oif eno1 accept
        iif eno1 oif vmbr0 accept
    }
}

table ip nat {
    chain prerouting {
        type nat hook prerouting priority -100;

        # Example DNAT (public port → VM)
        tcp dport 2222 dnat to 192.168.100.10:22    # VM1 SSH
        tcp dport 8080 dnat to 192.168.100.20:80    # VM2 Web
        tcp dport 5432 dnat to 192.168.100.30:5432  # VM3 DB
    }

    chain postrouting {
        type nat hook postrouting priority 100;

        # masquerade outbound traffic from VMs
        oif eno1 ip saddr 192.168.100.0/24 masquerade
    }
}
```

Apply and verify:

```bash
nft -f /etc/nftables.conf
nft list ruleset
```

---

## ⚙️ Step 4. Test Access

From outside:

- `ssh -p 2222 root@203.0.113.25` → VM1
- `curl http://203.0.113.25:8080` → VM2 webserver
- `psql -h 203.0.113.25 -p 5432 …` → VM3 database

---

## 🔒 Security Notes

- Only **one node can hold the public IP** at a time.
- Protect your Proxmox web interface (`8006`) — ideally allow only your IP or put it behind a VPN.
- Each VM must use a **unique external port**. If you want multiple websites, put an **nginx/traefik reverse proxy** VM behind the gateway and forward ports 80/443 only.
- If you want **automatic failover** (so if pve1 dies, pve2 takes the public IP), you’ll need **keepalived (VRRP)** for a floating IP setup.

---

## ✅ Summary

With one static public IP and two Proxmox nodes:

- Assign the public IP to one node (`pve1` or `pve2`).
- Configure the other node with only a private IP.
- Use **nftables** on the gateway node to forward traffic into the private subnet.
- Expose services via **unique ports** or a **reverse proxy**.

This way, you can run your whole cluster and still make services internet-accessible, even with just one public IP.
