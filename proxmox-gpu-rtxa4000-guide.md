# 🛠️ How to Set Up NVIDIA RTX A4000 GPU Passthrough in Proxmox VE for a Single AI VM

**By:** Cloud Ocampo  
**Updated:** July 2025

---

## 📌 Overview

This guide will walk you through the steps to **dedicate an NVIDIA RTX A4000 GPU** to a **single Ubuntu VM** in Proxmox VE 8.x using **VFIO passthrough**. You'll use this VM for GPU-intensive AI workloads like PyTorch, TensorFlow, or Jupyter notebooks.

---

## ⚙️ System Requirements

- **Proxmox VE Host** (Intel CPU with VT-d enabled)
- **NVIDIA RTX A4000** GPU installed on the host
- **Ubuntu 22.04 or 24.04 VM**
- **UEFI boot with IOMMU enabled**
- Access to the **Proxmox Shell** and VM configuration

---

## ✅ Step 1: Enable IOMMU on Proxmox Host

Edit your GRUB configuration:

```bash
nano /etc/default/grub
```

Find and update this line:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```

Then update GRUB:

```bash
update-grub
```

---

## ✅ Step 2: Load VFIO Modules

Edit or create this file:

```bash
nano /etc/modules
```

Add the following lines:

```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

---

## ✅ Step 3: Bind GPU to VFIO

Find your GPU device ID:

```bash
lspci -nnk | grep -iA3 nvidia
```

Example output:
```
5f:00.0 VGA compatible controller [0300]: NVIDIA GA104GL [RTX A4000] [10de:24b0]
5f:00.1 Audio device [0403]: NVIDIA High Definition Audio [10de:228b]
```

Now bind these to `vfio-pci`:

Create:
```bash
nano /etc/modprobe.d/vfio.conf
```

Add:
```
options vfio-pci ids=10de:24b0,10de:228b
```

---

## ✅ Step 4: Blacklist Nouveau (Open-source NVIDIA driver)

Create the blacklist file:

```bash
nano /etc/modprobe.d/blacklist-nouveau.conf
```

Add:
```
blacklist nouveau
options nouveau modeset=0
```

Then regenerate initramfs:

```bash
update-initramfs -u -k all
```

---

## 🔄 Step 5: Reboot

```bash
reboot
```

After reboot, verify GPU is now bound to VFIO:

```bash
lspci -nnk | grep -A3 -i nvidia
```

You should see:
```
Kernel driver in use: vfio-pci
```

---

## 🖥️ Step 6: Assign GPU to a VM in Proxmox

1. Shut down your target VM.
2. In the Proxmox GUI, go to your VM → **Hardware** → **Add → PCI Device**
3. Choose:
   - `0000:5f:00.0` (GPU)
   - `0000:5f:00.1` (HD Audio, optional)
4. **Check**:
   - “All Functions”
   - “Primary GPU” if this VM is headless
   - “PCI-Express”
   - “ROM-Bar” (optional)
5. Click **Add**, then **Start VM**

> 🛑 If you encounter "device assigned more than once" or `x-vga` errors, make sure you're not assigning the GPU to multiple VMs.

---

## 🧠 Step 7: Install NVIDIA Driver Inside VM

In the Ubuntu VM:

```bash
sudo apt update
sudo apt install -y nvidia-driver-535 nvidia-utils-535
```

Reboot:

```bash
sudo reboot
```

Verify:

```bash
nvidia-smi
```

You should see your RTX A4000 listed.

---

## 🚀 Optional: Create Python AI Environment

```bash
sudo apt install python3-venv python3-pip
python3 -m venv ai-env
source ai-env/bin/activate
pip install torch torchvision torchaudio
```

---

## 💡 BONUS: Run Jupyter on Boot via systemd

Create a systemd service:

```bash
nano ~/.config/systemd/user/jupyter.service
```

Add:
```ini
[Unit]
Description=Jupyter Notebook

[Service]
Type=simple
ExecStart=/home/youruser/ai-env/bin/jupyter notebook --no-browser --ip=0.0.0.0 --port=8888
WorkingDirectory=/home/youruser
Restart=always

[Install]
WantedBy=default.target
```

Enable and start:

```bash
systemctl --user daemon-reexec
systemctl --user enable jupyter
systemctl --user start jupyter
```

---

## ✅ Conclusion

You now have a fully functional AI-ready Ubuntu VM with **dedicated RTX A4000 GPU passthrough** on Proxmox VE. Perfect for:
- Model training
- Deep learning experiments
- CUDA-based computation
- Jupyter notebooks for AI dev
