# 🖥️ How to Add a New Storage Disk to a Proxmox VM (Example: File Server with 300GB Data Disk)

When running virtual machines (VMs) on **Proxmox**, the default VM setup usually has just one disk (the root/system disk). If you’re building a **file server** or need extra storage, you can easily add another disk (e.g., 300GB) dedicated to data.

This guide will walk you through adding, formatting, and mounting a new storage volume inside a VM.

---

## 1. Add a New Disk from Proxmox Web UI
1. Open the **Proxmox Web Interface**.  
2. Select your **VM** (e.g., `file-server`).  
3. Navigate to **Hardware → Add → Hard Disk**.  
4. Configure:
   - **Bus/Device**: `SCSI` (recommended for performance & hotplug support)  
   - **Storage**: Choose your Proxmox storage pool (e.g., `local-lvm` or `data-pool`)  
   - **Disk Size**: `300G`  
5. Click **Add**.

Your VM now has a second disk attached, but it is not yet formatted or mounted.

---

## 2. Verify Inside the VM
Log into your VM and check the disks:

```bash
lsblk
```

Expected output:
```
sda   64G
└─sda2   64G   /
sdb  300G        <-- New disk (unformatted)
```

The `sdb` device is your new 300GB disk.

---

## 3. Partition & Format the Disk
We’ll create a single partition and format it as **ext4**.

```bash
sudo fdisk /dev/sdb
```

Inside `fdisk`:
- Press `n` → create a **new partition**  
- Press `w` → **write changes**  

Then format the partition:
```bash
sudo mkfs.ext4 /dev/sdb1
```

---

## 4. Mount the Disk
Create a directory where you’ll mount the new disk (e.g., `/data`):

```bash
sudo mkdir -p /data
sudo mount /dev/sdb1 /data
```

Verify:
```bash
df -h
```
You should see:
```
/dev/sdb1   300G   1G  299G   1% /data
```

---

## 5. Make the Mount Permanent
By default, mounts are lost after reboot. To make it persistent, add an entry to `/etc/fstab`.

```bash
echo "/dev/sdb1 /data ext4 defaults 0 2" | sudo tee -a /etc/fstab
```

Reboot to confirm:
```bash
sudo reboot
```

After reboot:
```bash
df -h
```
You should still see `/data` mounted.

---

## ✅ Summary
Now your VM has:
- **System disk** (`/dev/sda`, e.g., 64GB, mounted at `/`)  
- **Data disk** (`/dev/sdb1`, 300GB, mounted at `/data`)  

You can use `/data` as a storage directory for:
- File uploads  
- Databases  
- Backups  
- Any application needing large storage  

---

⚡ Pro tip: Instead of `/dev/sdb1`, you can also use the disk’s **UUID** in `/etc/fstab` for safer mounting (prevents issues if device names change). Get it with:

```bash
blkid /dev/sdb1
```
