# 🗄️ How to Add Extra Disks as VM Storage in Proxmox VE After Installation

When you install **Proxmox VE** with ZFS RAID1, the installer will use the disks you select (e.g., `/dev/sda` and `/dev/sdb`) to create the `rpool` system pool. Any other disks in the server (like `/dev/sdc` and `/dev/sdd`) remain untouched and unallocated.  

The good news is you can still use those extra disks as dedicated storage for **VMs and containers** — without reinstalling Proxmox.  

This guide shows you how to do exactly that.  

---

## 🔹 Situation Example

- Proxmox installed on:  
  - `/dev/sda` (2.3 TB)  
  - `/dev/sdb` (2.3 TB)  
  - Configured as **ZFS RAID1 (`rpool`)**  

- Extra available disks:  
  - `/dev/sdc` (2.4 TB)  
  - `/dev/sdd` (2.4 TB)  
  - Currently unallocated  

Goal → Use `/dev/sdc` and `/dev/sdd` as **VM storage** in Proxmox.  

---

## 🔹 Step 1: Verify the Disks
Check your available disks and pools:  
```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT
zpool status
```

You should see `sda` + `sdb` in `rpool`, and `sdc` + `sdd` free (maybe with NTFS or no FS).  

---

## 🔹 Step 2: Wipe Old Partitions
Make sure the extra disks are clean:  
```bash
wipefs -a /dev/sdc
wipefs -a /dev/sdd
```

(optional: for a full wipe)  
```bash
sgdisk --zap-all /dev/sdc /dev/sdd
```

---

## 🔹 Step 3: Create a New ZFS Pool
Mirror the extra disks into a new pool named `vmdata`:  
```bash
zpool create vmdata mirror /dev/sdc /dev/sdd
```

Check the pool:  
```bash
zpool status
```

---

## 🔹 Step 4: Add the Pool to Proxmox
Tell Proxmox to use the new pool for VM images and containers:  
```bash
pvesm add zfspool vmdata -pool vmdata -content images,rootdir
```

Now if you run:  
```bash
pvesm status
```
You’ll see `vmdata` listed as active storage.  

---

## 🔹 Step 5: Use the Storage
- In the **Proxmox Web UI** → Datacenter → Storage, `vmdata` will appear.  
- When creating new VMs or LXCs, select **`vmdata`** as the target storage.  

---

## 🔹 Alternative Options
- **Max capacity (no redundancy):** use disks individually (`zpool create vmdata1 /dev/sdc`, etc.).  
- **Directory storage (ext4):** format a disk, mount it, then add with `pvesm add dir ...`.  

---

## ✅ Final Result
- **`rpool` (ZFS RAID1):** Proxmox system storage.  
- **`vmdata` (ZFS mirror):** Dedicated pool for VM and container storage.  
- Clean separation between system and workload disks.  

---

### Suggested Title
👉 **“How to Add Extra Disks as VM Storage in Proxmox VE After Installation”**
