
# 🚀 How to Create Proxmox VMs and Use Them in a Kubernetes Cluster (On-Premise Guide)

If you're running a powerful server using Proxmox VE and want to build a **production-ready Kubernetes cluster**, you're in the right place. This guide walks you through creating Ubuntu-based VMs in Proxmox and using them as **nodes (master and workers)** in a Kubernetes cluster — perfect for local development or internal production use.

---

## 🧱 Prerequisites

- A working Proxmox VE server (e.g., on Cisco UCS C240 M5 or any x86_64 server)
- Access to Proxmox web UI or CLI
- Ubuntu 22.04 ISO uploaded in Proxmox
- At least 3 static IP addresses (e.g., 192.168.1.100–102)
- Basic knowledge of Linux and networking

---

## 🖥️ Step 1: Create VMs in Proxmox for Kubernetes Nodes

### 🧰 VM Specs (minimum per node):
- **CPU**: 2–4 vCPUs
- **RAM**: 4–8 GB
- **Disk**: 20+ GB
- **Network**: Bridged (e.g., `vmbr0`) to LAN
- **Storage**: Local-lvm or ZFS

### 🔧 Steps to Create VM:
1. **Login to Proxmox web UI**
2. Click **Create VM**
3. Enter:
   - Node name: `k8s-master-1`, `k8s-worker-1`, etc.
   - ISO: `ubuntu-22.04-server.iso`
   - System: EFI BIOS, SCSI disk with `virtio-scsi`, QEMU agent enabled
4. **Hard Disk**: 20–40 GB (SSD-backed preferred)
5. **CPU**: 4 cores (host type)
6. **Memory**: 8192 MB (8 GB)
7. **Network**: Bridged to `vmbr0` (enable firewall if needed)
8. Finish and Start the VM

Repeat to create at least 3 VMs:
- `k8s-master-1`
- `k8s-worker-1`
- `k8s-worker-2`

---

## ⚙️ Step 2: Configure Ubuntu on Each VM

Install Ubuntu normally using your ISO. After setup:

```bash
sudo apt update && sudo apt upgrade -y
sudo hostnamectl set-hostname k8s-master-1   # (or worker-1 / worker-2)
```

Set static IP in `/etc/netplan/00-installer-config.yaml`:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: no
      addresses: [192.168.1.100/24]
      gateway4: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

Apply config:
```bash
sudo netplan apply
```

Install container runtime:
```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

---

## 🚀 Step 3: Install Kubernetes with RKE2 (Recommended for Production)

### 🟩 On Master Node:
```bash
curl -sfL https://get.rke2.io | sudo sh -
sudo systemctl enable rke2-server.service --now
```

Get node token:
```bash
sudo cat /var/lib/rancher/rke2/server/node-token
```

### 🟦 On Worker Nodes:
Create config `/etc/rancher/rke2/config.yaml`:
```yaml
server: https://192.168.1.100:9345
token: <paste-master-node-token>
```

Install:
```bash
curl -sfL https://get.rke2.io | sudo sh -
sudo systemctl enable rke2-agent.service --now
```

---

## 📦 Step 4: Connect and Verify the Cluster

On master:
```bash
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
kubectl get nodes
```

✅ You should see all nodes: master and workers in Ready status.

---

## 🧰 Optional Add-ons for Production

| Feature       | Tool                  |
|---------------|------------------------|
| Ingress       | NGINX Ingress Controller |
| TLS           | cert-manager            |
| Volumes       | Longhorn or hostPath    |
| Monitoring    | Prometheus + Grafana    |
| Dashboard     | Rancher UI              |
| Backup        | Velero, restic, snapshots |

---

## 🧠 Final Notes

- Proxmox + RKE2 gives you a **flexible, secure, on-prem Kubernetes environment**
- Snapshots, clones, and templates make managing nodes easy
- Can be extended with **HAProxy + Keepalived** for multi-master HA

Would you like a script version of this or Rancher added to the cluster? Let me know!

