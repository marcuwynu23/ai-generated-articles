
# ⚙️ How to Create Kubernetes Nodes and Schedule Pods Using Proxmox and Ubuntu VMs

This guide will walk you through how to:
- Create multiple **Ubuntu VMs in Proxmox**
- Use each VM as a **Kubernetes node**
- Deploy **pods across your cluster**
- Understand how **resources affect pod scheduling**

---

## 🧱 1. Setup Overview

### What You’ll Build:

```text
[ Proxmox Host ]
  ├─ VM 1: k8s-master-1 (Control Plane Node)
  ├─ VM 2: k8s-worker-1 (Worker Node)
  ├─ VM 3: k8s-worker-2 (Worker Node)
```

Each VM will run Ubuntu 22.04 with RKE2 or kubeadm installed.

---

## 🖥️ 2. Create VMs in Proxmox (Kubernetes Nodes)

### VM Specs per node:
- CPU: 2–4 vCPUs
- RAM: 4–8 GB
- Disk: 20–40 GB
- Network: Bridged (e.g., `vmbr0`)

Create 3 VMs in the Proxmox UI:
- `k8s-master-1`
- `k8s-worker-1`
- `k8s-worker-2`

Install Ubuntu Server 22.04 in each VM, and set a **static IP**.

---

## ⚙️ 3. Install RKE2 on All VMs

### On Master Node:
```bash
curl -sfL https://get.rke2.io | sudo sh -
sudo systemctl enable rke2-server.service --now
```

Get the token:
```bash
sudo cat /var/lib/rancher/rke2/server/node-token
```

### On Worker Nodes:
Create a file `/etc/rancher/rke2/config.yaml`:
```yaml
server: https://<master-ip>:9345
token: <paste-token>
```

Install RKE2 Agent:
```bash
curl -sfL https://get.rke2.io | sudo sh -
sudo systemctl enable rke2-agent.service --now
```

---

## 📋 4. Access Kubernetes Cluster

On the master node:
```bash
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
kubectl get nodes
```

✅ You should see:
```
NAME            STATUS   ROLES           AGE   VERSION
k8s-master-1    Ready    control-plane   5m    v1.28.x
k8s-worker-1    Ready    <none>          3m    v1.28.x
k8s-worker-2    Ready    <none>          2m    v1.28.x
```

---

## 📦 5. Deploy Pods and Observe Scheduling

### Example Deployment:
```bash
kubectl create deployment hello-world --image=nginx
kubectl scale deployment hello-world --replicas=10
```

### Check pod distribution:
```bash
kubectl get pods -o wide
```

You'll see pods spread across all worker nodes. Kubernetes schedules them **based on available CPU and RAM**.

---

## 📊 6. Pod Scheduling Logic

Pods get scheduled based on:
- Resource **requests** and **limits**
- Available **CPU/RAM** on each node
- Node **taints/labels** or **affinity/anti-affinity rules**

Example:
If a pod requests 500m CPU and 512Mi RAM, a node with 2 CPUs and 2GB RAM can run around 4 such pods.

---

## 🧠 Summary

- A **Proxmox VM** = 1 Kubernetes **Node**
- More nodes → more pods can be scheduled
- Use `kubectl get nodes`, `top`, `describe` to monitor usage
- You control scaling by **adding nodes** or **adjusting resources**

---

Would you like an automated script to spin up VMs and join them to the cluster?
