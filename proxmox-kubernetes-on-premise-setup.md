
# 🧾 How to Set Up an On-Premise Kubernetes Cluster Using Proxmox and K3s

## 🖥️ Step 1: Set Up Ubuntu Virtual Machines in Proxmox

### 🔧 1.1 Prepare Proxmox VE
- Download and install **Proxmox VE** from [https://www.proxmox.com](https://www.proxmox.com).
- Access the Proxmox web UI via `https://<proxmox-ip>:8006`.

### 🖥️ 1.2 Create VMs for Kubernetes
Create 4 VMs:
- **1 Master Node**: `k8s-master`
- **3 Worker Nodes**: `k8s-node-1`, `k8s-node-2`, `k8s-node-3`

Recommended Specs per VM:
- CPU: 2–4 cores
- RAM: 4–8 GB
- Disk: 20–40 GB
- OS: Ubuntu Server 24.04 LTS

## 🧾 Step 2: Set Hostnames and Hosts Mapping

### ✏️ 2.1 Set Unique Hostnames
```bash
sudo hostnamectl set-hostname k8s-master      # For master
sudo hostnamectl set-hostname k8s-node-1      # For workers
exec bash
```

### 🧭 2.2 Update `/etc/hosts` on All Nodes
```bash
sudo nano /etc/hosts
```
Add entries:
```
192.168.1.100  k8s-master
192.168.1.101  k8s-node-1
192.168.1.102  k8s-node-2
192.168.1.103  k8s-node-3
```

## 🔐 Step 3: Prepare All Nodes (Common Setup)

### 🔧 Disable Swap
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### 🧱 Enable Kernel Modules
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

### 📦 Install containerd
```bash
sudo apt update && sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

## 🌐 Step 4: Allow Required Ports (Firewall)

Open these on all nodes:
```bash
sudo ufw allow 22/tcp
sudo ufw allow 6443/tcp
sudo ufw allow 8472/udp
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
sudo ufw reload
sudo ufw enable
```

## 🚀 Step 5: Install K3s Kubernetes

### 🧭 On Master Node Only:
```bash
curl -sfL https://get.k3s.io | sh -
```

Check status:
```bash
sudo kubectl get nodes
```

Get join token:
```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

## 🧩 Step 6: Join Worker Nodes

On each worker node:
```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.100:6443 K3S_TOKEN=<PASTE_YOUR_TOKEN_HERE> sh -
```

## ✅ Step 7: Verify Cluster

```bash
sudo kubectl get nodes
```

## 🌐 Step 8: Deploy a Test Nginx App

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc
```

Access via:
```
http://<any-node-ip>:<nodePort>
```

## 📈 Step 9: Enable Autoscaling

Install metrics server:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Manual scale:
```bash
kubectl scale deployment nginx --replicas=3
```

Auto-scale:
```bash
kubectl autoscale deployment nginx --cpu-percent=50 --min=1 --max=5
```

## 📚 Summary

| Step | Action |
|------|--------|
| 🧱 VM Setup | Created 4 VMs via Proxmox |
| 🧾 Hostnames | Set unique names per node |
| 🔐 Firewall | Opened ports: 6443, 8472, 10250, 30000–32767 |
| 🚀 K3s Setup | Master with token, agents joined |
| 🧪 Testing | Deployed Nginx, exposed via NodePort |
| 📈 Autoscaling | Enabled metrics and HPA |
